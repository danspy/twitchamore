# TwitchAmore

A simple multi-stream Twitch viewer built with [Astro](https://astro.build).

## Features

- 🎮 Watch **multiple Twitch streams** simultaneously in a responsive grid
- 🔊 **Mute / unmute** individual streams at a glance from the control bar or stream overlay
- ⬛ **Stage mode** – feature one stream prominently (75 %) while the others stack in a sidebar
- 🔐 **Twitch login** (optional) – sign in to show your display name & avatar via Twitch OAuth
- 💾 **Persists** your stream list, mute states, and staged stream across browser sessions

## Screenshots

**Multi-stream grid view**

![Multi-stream grid](https://github.com/user-attachments/assets/f72be0ea-7f1e-44ea-bb24-24101ea8bbe7)

**Stage mode** – ninja is featured, shroud is muted in the sidebar

![Stage mode](https://github.com/user-attachments/assets/4bfce743-b4db-4e34-8fd1-fadaeed336cf)

## Tech stack

| Layer | Choice |
|-------|--------|
| Framework | [Astro](https://astro.build) v5 (static output) |
| Stream embeds | [Twitch Embed API](https://dev.twitch.tv/docs/embed/everything/) |
| Authentication | Twitch OAuth 2.0 implicit-grant flow |
| Styling | Vanilla CSS with Twitch-inspired dark theme |
| State | `localStorage` (no backend required) |

## Setup

### Prerequisites

- Node.js 18+

### 1. Install dependencies

```bash
npm install
```

### 2. (Optional) Configure Twitch login

Streams work **without** login. To enable the "Login with Twitch" button:

1. Go to the [Twitch Developer Console](https://dev.twitch.tv/console/apps) and create a new application.
2. Add your redirect URL to **OAuth Redirect URLs**:
   - Development: `http://localhost:4321`
   - Production: your deployed URL
3. Copy the **Client ID**.
4. Create a `.env` file (see `.env.example`):

```env
PUBLIC_TWITCH_CLIENT_ID=your_client_id_here
```

### 3. Start the development server

```bash
npm run dev
```

Open [http://localhost:4321](http://localhost:4321).

## Usage

| Action | How |
|--------|-----|
| Add a stream | Type a Twitch username in the header and press **Enter** or click **Add Stream** |
| Remove a stream | Hover over the stream → click **✕**, or click **✕** in the control bar |
| Mute / unmute | Hover over the stream → click **🔊/🔇**, or click in the control bar |
| Stage a stream | Hover over the stream → click **⬛**, or click in the control bar. Click again to exit stage mode |
| Login with Twitch | Click **Login with Twitch** in the header (requires `PUBLIC_TWITCH_CLIENT_ID`) |

## Build

```bash
npm run build      # outputs to dist/
npm run preview    # preview the production build locally
```
