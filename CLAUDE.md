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

TwitchAmore is a static single-page app built with Astro v5. The entire application lives in **one file**: `src/pages/index.astro` (~2435 lines).

That single file contains:
- All HTML markup
- All CSS (using `<style is:global>` — Twitch-inspired dark theme with CSS custom properties in `:root`)
- All client-side JavaScript (in a single `<script>` tag — vanilla JS, no framework)

The Twitch Embed SDK (`embed.twitch.tv/embed/v1.js`) is loaded as a classic inline script so `window.Twitch` is available globally.

### File structure

```
src/pages/index.astro   # entire application
src/env.d.ts            # TypeScript env var declarations
astro.config.mjs        # Astro config (static output, BASE_PATH support)
public/favicon.svg      # Twitch-purple favicon
.env.example            # template for optional OAuth env var
```

### Client-side state

All mutable state lives in module-level variables:

| Variable | Type | Description |
|---|---|---|
| `streams` | `string[]` | Ordered list of channel names (always lowercase) |
| `stageChannel` | `string \| null` | Currently staged (featured) channel |
| `mutedSet` | `Set<string>` | Channels that are muted |
| `embeds` | `{ [ch]: { embed, player } }` | Twitch embed instances |
| `dragSrc` | `string \| null` | Channel being dragged in the control bar |
| `layoutMode` | `'stage' \| 'grid'` | Desktop layout mode |
| `stageFullWidth` | `boolean` | Full-width override for staged stream |
| `chatOpen` | `boolean` | Chat sidebar visibility |
| `chatChannel` | `string \| null` | Channel shown in chat sidebar |
| `chatPinned` | `boolean` | Whether sidebar is pinned beside streams |
| `theaterMode` | `boolean` | Hides header/footer for immersive full-screen viewing |
| `twitchToken` | `string \| null` | OAuth 2.0 access token |
| `twitchUser` | `object \| null` | `{ id, login, displayName, profileImageUrl }` |
| `followedLive` | `array` | Live streams from the authenticated user's followed channels |

State is persisted to `localStorage` on every mutation via `save()` and restored on load via `load()`. Keys: `ta_streams`, `ta_muted`, `ta_stage`, `ta_layout`, `ta_stage_fullwidth`, `ta_chat_open`, `ta_chat_pinned`, `ta_chat_ch`, `ta_token`, `ta_user`.

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

### Key functions

| Function | Description |
|---|---|
| `save()` / `load()` | Persist/restore all state to/from localStorage |
| `addStream(raw)` | Add a channel (trim, lowercase, deduplicate) |
| `removeStream(ch)` | Remove channel and destroy its embed |
| `clearAllStreams()` | Remove all streams at once |
| `toggleMute(ch)` / `setMuted(ch, bool)` | Mute state management |
| `setStage(ch)` | Toggle stage/primary stream; auto-mutes others |
| `toggleStageFullWidth(ch)` | Full-width override for staged stream |
| `applyLayout()` | Central layout computation and DOM update |
| `initEmbed(ch)` | Create `Twitch.Embed` instance; apply mute in `VIDEO_READY` |
| `createCard(ch)` | Create `.stream-card` DOM element with overlay buttons |
| `renderBar()` | Render footer control bar with draggable stream items |
| `toggleLayoutMode()` | Switch `'stage'` ↔ `'grid'` (desktop only) |
| `toggleTheater()` | Hide/show header+footer for immersive viewing |
| `muteAll()` | Mute all streams (or unmute all if all are muted) |
| `openChatSidebar()` / `closeChatSidebar()` | Chat sidebar visibility |
| `setChatChannel(ch)` | Switch which stream's chat is shown |
| `toggleChatPin()` | Pin sidebar beside streams (tablet+ only) |
| `setupChannelSuggest(inputId, listId, onPick)` | Autocomplete with GQL typeahead; debounced 220ms |
| `loginWithTwitch()` | Initiate OAuth 2.0 implicit-grant flow |
| `logout()` | Revoke token, clear session |
| `fetchFollowedStreams()` | Fetch live followed channels via Helix API |
| `exportLayout()` / `importLayout(file)` | Serialize/restore layout as JSON |
| `escHtml(str)` | HTML-escape helper — use for any user-supplied content inserted into DOM |

### CSS theme

Custom properties defined in `:root`:

```css
--bg:          #0e0e10    /* Main background */
--bg-header:   #18181b    /* Header / control bar */
--bg-card:     #1f1f23    /* Cards / inputs */
--border:      #2d2d35    /* Borders */
--accent:      #9147ff    /* Twitch purple */
--accent-hov:  #a970ff    /* Hover purple */
--text:        #efeff1    /* Primary text */
--text-dim:    #adadb8    /* Secondary text */
--danger:      #e91916    /* LIVE badges / destructive actions */
--header-h:    58px
--bar-h:       52px
```

### Twitch integration

- **Embeds**: `new Twitch.Embed(containerId, options)` — always start muted to satisfy browser autoplay policy; real mute state applied in `VIDEO_READY` event.
- **Channel autocomplete**: Fetches from `https://gql.twitch.tv/gql` using the public Twitch GQL client ID (`kimne78kx3ncx6brgo4mv6wki5h1ko`) and the `ChannelTypeahead` operation. Debounced 220ms. Followed live streams appear first in results.
- **Auth (optional)**: Twitch OAuth 2.0 implicit-grant flow. Requires `PUBLIC_TWITCH_CLIENT_ID` env var (defined in `src/env.d.ts`). Without it the login button is hidden; streams still work. Scope: `user:read:follows`.
- **Followed streams**: Fetched via `/helix/streams/followed` when logged in. Auto-refreshes every 2 minutes. Shown in empty state and prioritized in autocomplete.

### Environment

Create `.env` for optional Twitch login:
```
PUBLIC_TWITCH_CLIENT_ID=your_client_id_here
```

The `BASE_PATH` env var is read by `astro.config.mjs` for deployment to a sub-path.

### Conventions

- **Channel names are always stored and compared lowercase.** Normalize with `.trim().toLowerCase()` before any lookup.
- **Never insert unsanitized strings into innerHTML.** Use `escHtml()` for any user-supplied content.
- **`applyLayout()` is the source of truth for the DOM grid.** Don't manipulate `.stream-card` grid placement anywhere else.
- **Embeds are created in `applyLayout()`, not `createCard()`.** `createCard()` only produces the DOM shell.
- **All state mutations must call `save()`.** This keeps localStorage in sync.
- **`theaterMode` hides `#header` and `#ctrl-bar`** by toggling a `.theater` class on `<body>`.
