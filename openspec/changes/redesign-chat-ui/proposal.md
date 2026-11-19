# Proposal

## Why

The app is a fork of an upstream general-purpose ChatGPT front end and still reads like a developer demo: an inherited author credit sits under every chat, the left rail is a fixed-width list of `Chat 1`/`Chat 2`, and messages are undifferentiated Bulma tiles with no speaker identity. It is being used as a private character/roleplay client, so the interface needs to feel like one — a composed, quiet reading surface built around the characters in the chat — and the chat list needs to stop eating a third of the screen while the reader is actually in a conversation.

## What Changes

- Remove the footer credit ("ChatGPT-web by Niek van der Maas" + GitHub icon) from every page that renders it. The layout container stays for the fixed-input masking behaviour.
- Rework the visual design into a character-chat reading surface: a dark-first token system, a serif prose face for message bodies, message rows that carry speaker identity (avatar initial + character name) instead of anonymous coloured tiles, a centered reading column, and a composed floating composer.
- Make the left chat list collapsible — a toggle in the rail header collapses it to zero width so the conversation gets the full window, and the collapsed state persists across reloads.
- Auto-name a chat from the model on the first exchange: once the first assistant reply lands, a chat that still carries its default `Chat N` name is renamed from a short model-generated summary of the conversation, without touching the message history or blocking the UI.
- Keep manual authority: a name the user has typed or renamed wins permanently, and the existing "suggest name" action stays available.

**BREAKING**: none at the API/URL level. `Chat` records gain two optional fields (`nameEdited`, `autoNamed`); old records without them are treated as not-yet-auto-named and not manually edited, so existing chats keep their names.

## Capabilities

### New Capabilities

- `chat-ui`: The application shell and conversation surface — layout, theming, the collapsible chat rail, message presentation with speaker identity, the composer, and the absence of upstream author credit.
- `chat-titling`: How a chat acquires its display name — the default `Chat N` seed, model-generated naming on the first exchange, and the user's manual name taking precedence.

### Modified Capabilities

None. This is the first spec set for the fork.

## Impact

- `src/app.scss` (token system and component styling), `src/lib/Footer.svelte` (credit removed), `src/lib/Sidebar.svelte` + `src/App.svelte` (collapse), `src/lib/Chat.svelte` (auto-title trigger, header), `src/lib/EditMessage.svelte` (speaker identity row), `src/lib/Home.svelte` (setup screen), `src/lib/Storage.svelte` (persisted collapse state), `src/lib/Settings.svelte` / `src/lib/Types.svelte` (small helpers and optional fields).
- No new runtime dependencies; no network calls added (the auto-title request reuses the existing chat-completion path and the user's own provider key).
- Deploy path unchanged: push to `main` → GitHub Pages.
- Privacy posture unchanged: no data leaves the browser except the completion requests the user already sends to their configured provider. The auto-title call goes to the same provider endpoint.
