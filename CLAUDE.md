# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # start dev server at http://localhost:4321
npm run build    # build static output to dist/
npm run preview  # preview the production build locally
```

There are no tests or linters configured.

## Architecture

TwitchAmore is a static single-page app built with Astro v5. The entire application lives in **one file**: `src/pages/index.astro`.

That single file contains:
- All HTML markup
- All CSS (using `<style is:global>` — Twitch-inspired dark theme with CSS custom properties in `:root`)
- All client-side JavaScript (in a single `<script>` tag — vanilla JS, no framework)

The Twitch Embed SDK (`embed.twitch.tv/embed/v1.js`) is loaded as a classic inline script so `window.Twitch` is available globally.

### Client-side state

All mutable state lives in module-level variables:

| Variable | Type | Description |
|---|---|---|
| `streams` | `string[]` | Ordered list of channel names |
| `stageChannel` | `string \| null` | Currently staged (featured) channel |
| `mutedSet` | `Set<string>` | Channels that are muted |
| `embeds` | `{ [ch]: { embed, player } }` | Twitch embed instances |
| `layoutMode` | `'stage' \| 'grid'` | Desktop layout mode |
| `stageFullWidth` | `boolean` | Full-width override for staged stream |
| `chatOpen` | `boolean` | Chat sidebar visibility |
| `chatChannel` | `string \| null` | Channel shown in chat sidebar |
| `chatPinned` | `boolean` | Whether sidebar is pinned beside streams |

State is persisted to `localStorage` on every mutation via `save()` and restored on load via `load()`. Keys: `ta_streams`, `ta_muted`, `ta_stage`, `ta_layout`, `ta_stage_fullwidth`, `ta_chat_open`, `ta_chat_pinned`, `ta_chat_ch`.

### Layout engine

`applyLayout()` is the central function that computes and applies the responsive grid layout based on:
- `streams.length`
- `stageChannel` / `stageFullWidth`
- `layoutMode` (`'stage'` or `'grid'`)
- Container width (`container.clientWidth`)

It sets CSS grid properties directly on `#streams` and individual `.stream-card` elements. It is triggered on state changes and by a `ResizeObserver` on the streams container.

Layout breakpoints:
- **<600px** (mobile): single column, 16:9 aspect-ratio cards
- **600–899px** (tablet): 2-col grid; staged stream spans full width at top row
- **≥900px** (desktop): `'stage'` mode = `wide-stage` layout (8-col primary + 4-col sidebar); `'grid'` mode = 2-col grid

Embeds are initialized inside `applyLayout()` (via `initEmbed(channel)`) after grid dimensions are set — not in `createCard()`.

### Twitch integration

- **Embeds**: `new Twitch.Embed(containerId, options)` — always start muted to satisfy browser autoplay policy; real mute state applied in `VIDEO_READY` event.
- **Channel autocomplete**: Fetches from `https://gql.twitch.tv/gql` using the public Twitch GQL client ID (`kimne78kx3ncx6brgo4mv6wki5h1ko`) and the `ChannelTypeahead` operation. Debounced 220ms.
- **Auth (optional)**: Twitch OAuth 2.0 implicit-grant flow. Requires `PUBLIC_TWITCH_CLIENT_ID` env var (defined in `src/env.d.ts`). Without it the login button is hidden; streams still work.

### Environment

Create `.env` for optional Twitch login:
```
PUBLIC_TWITCH_CLIENT_ID=your_client_id_here
```

The `BASE_PATH` env var is read by `astro.config.mjs` for deployment to a sub-path.
