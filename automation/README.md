# 🤖 Perfect Setup — Automation module

Guided web app that automates the manual steps in the guide: create/use a **Stremio** (later
**Nuvio**) account, build the **AIOStreams** / **AIOMetadata** / **Watchly** configs from the repo
templates, install everything in the right order, and hand back every credential created.

> **Status: Phase 1 (Stremio MVP) scaffold.** The template engine + Stremio + AIOStreams paths are
> implemented and unit-tested offline. AIOMetadata, Watchly, Nuvio, Trakt, and the Cloudflare
> Worker proxy are stubbed/flagged for later phases. See [`../AUTOMATION-PLAN.md`](../AUTOMATION-PLAN.md)
> and [`../API-NOTES.md`](../API-NOTES.md).

## Layout

```
automation/
  config.example.json        Parameterized config (instances, fallbacks, preferences)
  src/
    template-engine.js        Ports the AIOStreams directive engine (__if/__switch/{{inputs}})
    schema-renderer.js        Builds the wizard form from template.metadata.inputs (dynamic UI)
    orchestrator.js           Phase-1 Stremio flow (account -> aiostreams -> install)
    adapters/
      stremio.js              api.strem.io: login/register, addonCollectionGet/Set, ordering + Cinemeta patch
      aiostreams.js           POST /api/v1/user -> {uuid, encryptedPassword, manifestUrl}
  ui/
    index.html, app.js        Static wizard, deployable to GitHub Pages
  test/
    template-engine.test.mjs  Offline tests against the REAL templates/AIOStreams.json
```

## Why this design

- **The UI is generated from the template.** `templates/AIOStreams.json` carries a self-describing
  form schema in `metadata.inputs`. `schema-renderer.js` renders it and `template-engine.js`
  resolves the result — so **editing the template changes the interface automatically**, no UI code
  change needed.
- **AIOStreams has open CORS** (`Access-Control-Allow-Origin: *`, confirmed in upstream source), so
  the wizard can call it straight from GitHub Pages. **Stremio's `api.strem.io`** is browser-callable
  too and a single `addonCollectionSet` does install + ordering + Cinemeta clean-up (replacing
  Cinebye). Only **Trakt** strictly needs the Cloudflare Worker proxy (no CORS) — Phase 2.

## Run the tests (no network needed)

```bash
node automation/test/template-engine.test.mjs
```

## Try the wizard locally

```bash
# serve the repo root so ui/ can fetch ../templates and ../src
python3 -m http.server 8000
# open http://localhost:8000/automation/ui/
```

Copy `config.example.json` to `config.json` to point at your preferred instances / fallbacks.
API keys and passwords are entered in the wizard at runtime and are **never** written to disk.

## Deploy to GitHub Pages

Publish `automation/ui/` (it fetches `../src/*` and the templates by raw URL). The current Pages
build serves from `docs/`; either add an `automation/` include to the Pages workflow or host the
`ui/` folder as a separate Pages site / project page.

## Roadmap (see AUTOMATION-PLAN.md §8)

- **Phase 1 (here):** Stremio account + AIOStreams create & install, dynamic form, credential summary.
- **Phase 2:** AIOMetadata save + install, Trakt device OAuth via Worker, Watchly, Watch Next.
- **Phase 3:** Nuvio (account/profiles/addons/collections), multi-instance Autopilot fallback.
- **Phase 4:** Resumability, error surfacing, local CLI mode, template-version compatibility guard.
