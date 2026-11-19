# Design

## Context

See `proposal.md` — Why. The relevant current state:

- Layout is a two-column flex shell: `App.svelte` renders `<Navbar />`, a `.side-bar-column` holding `Sidebar.svelte`, and a `.main-content-column`. Width is driven by two CSS variables on `html`: `--sidebarWidth: max(300px, 20%)` and `--mainContentWidth: calc(100% - var(--sidebarWidth))`.
- Styling is one file, `src/app.scss`, importing full Bulma and then patching layout with hand-written rules. Light/dark come from `@media (prefers-color-scheme: dark)` flipping three variables (`--BgColor`, `--BgColorSidebar*`, `--thumbBG`).
- Messages are `article.message.chat-message` with Bulma colour modifiers (`is-info` user, `is-success` assistant, `is-warning` system). There is no speaker header; `EditMessage.svelte` renders `.message-body > .message-display`.
- `Footer.svelte` renders a fixed container (also used for the composer's mask), and unconditionally prints the upstream credit block inside it.
- Chat names are seeded in `Storage.svelte` (`addChat` → `newName('Chat ${chatId}', ...)`). `Chat.svelte` already contains `suggestName()`, a manual action wired to a lightbulb button that sends a summary request via `ChatRequest.sendRequest(..., { summaryRequest: true, autoAddMessages: false })` and writes the reply to `chat.name`.
- `ChatSettings` already carries `characterName` (default `'ChatGPT'`) and `profileName` — the data needed for speaker identity exists.
- Persistence helper `persisted(key, default)` from `svelte-local-storage-store` is already used for every store in `Storage.svelte`.

## Goals / Non-Goals

**Goals:**
- A composed reading surface suitable for character/roleplay chat, without replacing the component tree wholesale — Bulma primitives stay; the visual layer and a few structural additions change.
- Collapse behaviour that costs one boolean and one CSS variable, not a second layout.
- Auto-naming that reuses the existing summary request path — no new client code for talking to providers.

**Non-Goals:**
- No character cards, avatars-as-images, group chats, or SillyTavern import. Speaker identity is a name plus an initial.
- No in-app theme picker; system colour scheme only.
- No change to the settings modal, profile system, or chat storage format beyond two optional boolean fields.
- No touch to `src-tauri`, Docker, or the CI workflow.

## Decisions

**1. Token layer in `app.scss`, layered over Bulma rather than forking it.**
Define a semantic set of CSS custom properties on `html` — `--surface-base`, `--surface-rail`, `--surface-card`, `--border-soft`, `--text-strong`, `--text-muted`, `--accent`, `--accent-soft`, `--prose` — and set both palettes in one `@media (prefers-color-scheme)` pair. Components then stop carrying Bulma colour classes in CSS and read the tokens. Reason: re-theming a Bulma app by overriding `is-info`/`is-success` individually is a whack-a-mole that leaves stray hues; a token layer makes every override resolve from one place. Alternative rejected: adopting another CSS framework (Tailwind/Pico) — a rewrite of every component for a cosmetic outcome.

**2. Dark-first identity: ink and brass.**
Dark palette anchors on a warm near-black with a plum undertone (`#12111a`), a slightly lifted rail (`#17161f`), card surface (`#1c1b26`), and a muted brass accent (`#d3ae72`). Light palette is unbleached paper (`#f8f5ef`) with a darker bronze accent for contrast (`#8a6a2c`). Reason: the brief is character chat — warm metal on ink reads as a reading lamp and a book, distinct from the two generated-design defaults (acid green on near-black; terracotta on cream). The accent appears in exactly three places: the active chat marker, the character name, and the send action.

**3. Serif prose, sans chrome.**
Message bodies render in a system serif stack (`Charter, 'Bitstream Charter', Georgia, serif`) at 17px/1.7; all UI (rail, labels, buttons, inputs) stays in the system sans stack. Reason: prose in a serif is the single cheapest change that makes an interface read as a reading surface rather than an admin panel, and a system stack avoids adding a webfont — which would be a new network request and a privacy regression for an app that advertises "nothing leaves the browser". Alternative rejected: bundling a webfont (adds ~40–120 KB and a build step for no behavioural gain).

**4. Speaker identity derived from existing settings, not new data.**
`EditMessage.svelte` gains a header row for non-system messages: an initial-letter mark (first character of `characterName`, uppercased) plus the name for assistant turns, and a right-aligned "You" label for user turns. Fallback chain: `characterName` → `'Assistant'`. No new stored fields, so no migration.

**5. Collapse as a CSS variable plus a persisted store.**
Add `sidebarCollapsed = persisted('sidebarCollapsed', false)` to `Storage.svelte`. `App.svelte` applies `class:rail-collapsed={$sidebarCollapsed}` on `#app`; `app.scss` sets `--sidebarWidth: 0px` under that class and hides the rail, with a fixed-position toggle button that stays visible in both states. The rail's own header also carries the toggle. Reason: zero-layout cost, no second markup tree, and the transition is a single width animation. Alternative rejected: an off-canvas drawer with overlay — more correct on mobile but adds an overlay element, focus trapping, and a body-scroll lock for a desktop-first app.

**6. Auto-naming reuses `suggestName()` with an auto flag.**
Refactor `suggestName(auto = false)`: when `auto`, skip the error message and skip the `updatingMessage` banner, then invoke it from `submitForm` after `await response.promiseToFinish()` when all of these hold: the chat has at least one assistant message, `chat.autoNamed !== true`, `chat.nameEdited !== true`, and the name still matches `/^Chat \d+$/`. Set `chat.autoNamed = true` and persist *before* the request so a failure or a reload cannot loop it. Manual rename paths (`promptRename`, `ChatMenuItem` edit) set `chat.nameEdited = true`.
Reason: the summary path already exists, already uses `summaryRequest: true` (so it bypasses image generation, continuous-chat summarisation, and message auto-add), and records no usage against the transcript. Alternative rejected: a separate tiny completion call written from scratch — duplicate request/streaming/abort handling for no gain.

**7. Footer credit removed at the source, container kept.**
`Footer.svelte` keeps its fixed masks and `.section-footer` wrapper (the composer's gradient mask and the `.pin-footer` positioning depend on them) and drops the credit block entirely. Reason: the credit is presentational, but the container is load-bearing for the fixed composer.

## Risks / Trade-offs

- **[Auto-naming spends the reader's tokens on a request they did not type]** → One request, only on the first exchange, only when the name is untouched, `maxTokens` bounded by the summary path; a failure is silent and never retried. Rough cost: a few hundred tokens once per chat.
- **[A slow provider makes the first reply appear to finish before the name changes]** → Naming is fire-and-forget after the reply; the seed name remains until it resolves, and the sidebar re-renders on `$checkStateChange`.
- **[Token-based theming leaves untouched Bulma components (settings modal) half-themed]** → The token layer also overrides Bulma's own CSS variables (`--bulma-*`) for surface, border, text, and radius, so components that read Bulma variables inherit the palette without per-component rules.
- **[Collapsed rail hides the only "New chat" affordance]** → The floating toggle restores it in one click, and the rail header toggle remains available whenever the rail is open; no state is lost.
- **[Serif prose changes line-length perception on narrow screens]** → Reading measure caps at 46rem and drops to full width under 768px, where padding already tightens.
- **[Existing chats losing names to auto-naming]** → Guard requires the literal seed pattern, `autoNamed !== true`, and `nameEdited !== true`, so any chat already renamed — or already carrying messages, since the trigger is the first exchange — is out of scope.

## Migration Plan

- Deploy is the existing GitHub Pages workflow: commit to `main`, the workflow builds `--base=/chatgpt-web/` and publishes `gh-pages`.
- Stored data needs no migration: `nameEdited` and `autoNamed` are optional, absent means "not set". Bumping the localStorage schema is unnecessary.
- Rollback: `git revert` the merge commit and push; the deployed bundle reverts and stored chats are unaffected (no keys were renamed or removed).

## Open Questions

None.
