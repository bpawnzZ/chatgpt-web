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

## Verify a deploy
```bash
gh run list --repo bpawnzZ/chatgpt-web --limit 3
curl -s -o /dev/null -w '%{http_code}\n' https://bpawnzZ.github.io/chatgpt-web/
```
