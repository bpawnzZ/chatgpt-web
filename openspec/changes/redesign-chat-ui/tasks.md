# Tasks

## 1. Theme foundation

- [x] 1.1 Add the semantic token block to `src/app.scss` (`--surface-base`, `--surface-rail`, `--surface-card`, `--border-soft`, `--text-strong`, `--text-muted`, `--accent`, `--accent-soft`, `--prose`) for both light and dark, and map Bulma's own `--bulma-*` surface/border/text/radius variables onto them. Verify: `npm run build` succeeds and the built CSS contains both palettes (`grep -c 'surface-card' dist/assets/*.css`).
- [x] 1.2 Remove the upstream credit block (author text, personal-site link, GitHub icon) from `src/lib/Footer.svelte` while keeping the fixed mask/container elements. Verify: no `niekvandermaas.nl` or `faGithub` reference remains in `src/lib/Footer.svelte` or the built bundle.
- [x] 1.3 Establish the type scale and reading measure: serif prose stack for message bodies, sans for chrome, 46rem centred content column. Verify: rule present in built CSS and a long message wraps inside the measure.

## 2. Shell and collapsible rail

- [x] 2.1 Add `sidebarCollapsed = persisted('sidebarCollapsed', false)` to `src/lib/Storage.svelte`. Verify: store exports and typechecks (`npm run check`).
- [x] 2.2 Wire the collapse class in `src/App.svelte` and drive `--sidebarWidth` from it in `app.scss`, with a fixed toggle control visible in both states. Verify: clicking the toggle collapses the rail to 0 width and the toggle stays clickable.
- [x] 2.3 Add the rail-header toggle and the collapse-on-navigate behaviour for narrow viewports in `src/lib/Sidebar.svelte` / `src/lib/ChatMenuItem.svelte`. Verify: on a ≤768px viewport, selecting a chat closes the rail; the collapsed state survives a reload.

## 3. Conversation surface

- [x] 3.1 Restyle the message rows in `src/app.scss` — card surface, soft border, brass left rule for assistant turns, muted raised surface for user turns, remove the Bulma colour modifiers' influence on message fill. Verify: user and assistant turns are visually distinct and legible in both schemes.
- [x] 3.2 Add the speaker header to `src/lib/EditMessage.svelte`: initial mark plus `chat.settings.characterName` (fallback `Assistant`) for assistant turns, "You" label for user turns; hidden for system, error, and image-only messages. Verify: an assistant message shows the character name and an initial; a chat with no character name shows `Assistant`.
- [x] 3.3 Restyle the composer and chat header in `src/app.scss` / `src/lib/Chat.svelte` as a floating rounded panel aligned to the reading column, keeping Enter-to-send, Shift+Enter newline, auto-grow, and the send/cancel/mic/settings/queue controls. Verify: composer stays visible while scrolling a long chat and grows on multi-line input.
- [x] 3.4 Restyle the setup screen (`src/lib/Home.svelte`) as a centred connect card. Verify: provider select, key field, custom base URL, and Test & save still function with no API key configured.
- [x] 3.5 Add roleplay formatting: split quoted spans out of markdown text nodes and colour narration (emphasis) against dialogue in both schemes. Verify: a message mixing `*action*` and "speech" shows three distinct text colours and the quotes are preserved.

## 4. Auto-titling

- [x] 4.1 Add optional `nameEdited?: boolean` and `autoNamed?: boolean` to `Chat` in `src/lib/Types.svelte`. Verify: `npm run check` passes.
- [x] 4.2 Refactor `suggestName()` in `src/lib/Chat.svelte` into an auto-aware form that suppresses the error message and progress banner when invoked automatically. Verify: manual "suggest name" still inserts its error message on failure; the auto path inserts nothing.
- [x] 4.3 Trigger auto-naming from `submitForm` after the first assistant reply, guarded by seed-name pattern, `!chat.autoNamed`, `!chat.nameEdited`, marking `autoNamed` and persisting before the request. Verify: a fresh chat's name changes to a topic label after the first reply; the transcript gains no extra message.
- [x] 4.4 Mark manual renaming in `promptRename` (`Chat.svelte`) and the sidebar editor (`ChatMenuItem.svelte`) by setting `nameEdited`. Verify: renaming a chat before the first reply completes prevents auto-naming for that chat.

## 5. Integration checks

- [x] 5.1 `npm run check` and `npm run lint` pass with no new errors.
- [x] 5.2 `npm run build:github` succeeds and the local preview renders the setup screen, a chat with messages, and both rail states.
- [x] 5.3 Confirm no new network origins: only the configured provider endpoint is contacted (the naming request included), and chats/keys remain in `localStorage` only.
- [ ] 5.4 Commit and push to `main`; confirm the Pages workflow succeeds and the live URL renders the redesign.

## Workflow follow-up

- Run `openspec archive redesign-chat-ui` once the deployed change is confirmed.
