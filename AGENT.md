# AGENT.md — chatgpt-web (fork of Niek/chatgpt-web)

Static Svelte SPA. No backend, no server-side secrets: chats + API keys live in
browser localStorage (Dexie/IndexedDB for messages), prompts go straight from the
browser to whichever provider you pick.

## Live URL
https://bpawnzZ.github.io/chatgpt-web/

## Deploy = push to main
`.github/workflows/pages.yml` runs `npm ci && npm run lint && npm run build:github`
(`vite build --base=/chatgpt-web/`) on every push to `main`, then publishes `dist`
to the `gh-pages` branch with peaceiris/actions-gh-pages. Pages serves `gh-pages`
(root). So: **push to main → live in ~1 min.** No manual deploy step.

The `--base=/chatgpt-web/` flag is required — Pages serves under a subpath and a
root-base build 404s every asset.

## Local sanity check (optional)
```bash
npm ci
npm run build:github   # what CI builds
npm run dev            # http://localhost:5173 (root base)
```

## Config
`.env` is committed but every var is commented out — defaults mean "let the user
pick a provider and paste their own key in the settings screen". Don't uncomment
`VITE_API_KEY` in a repo you publish: it gets baked into the bundle.

## UI layer (this fork's redesign)

Planning lives in `openspec/` — `openspec list --specs`, and past changes under
`openspec/changes/archive/`. Specs: `chat-ui` (shell, rail, messages, formatting)
and `chat-titling` (how a chat gets its name).

- All theming is one token block at the top of `src/app.scss`
  (`--surface-*`, `--border-soft`, `--text-*`, `--accent`, `--speech`, `--narration`,
  `--reading-measure`), light values in `html`, dark overrides in the
  `prefers-color-scheme` block. Change the palette there, not per component.
- Message bodies render in a serif stack; chrome stays sans. Emphasis (`*action*`)
  is narration-coloured via CSS; quoted spans are split out by
  `src/lib/SpeechText.svelte`, registered as the `text` markdown renderer in
  `EditMessage.svelte`.
- Rail collapse is `sidebarCollapsed` in `Storage.svelte` + `html.rail-collapsed`
  (`--sidebarWidth: 0`). Desktop-only; phones keep the legacy navbar pin.
- Chats auto-name once after the first assistant reply (`autoNameChat` in
  `Chat.svelte`): guarded by `/^Chat \d+$/`, `!autoNamed`, `!nameEdited`, on a
  dedicated `ChatRequest` so it can't collide with a message being sent.

## Verify a deploy
```bash
gh run list --repo bpawnzZ/chatgpt-web --limit 3
curl -s -o /dev/null -w '%{http_code}\n' https://bpawnzZ.github.io/chatgpt-web/
```
