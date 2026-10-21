# CLAUDE.md

Technical guide for Claude Code in this project. What the product does is in [README.md](README.md); the details are in [docs/architecture.md](docs/architecture.md) and [docs/screens.md](docs/screens.md).

## What this is

A Google Apps Script web app, bound to a Google Sheet, that uses **clasp** to sync code with Google and **Fuelix** (TELUS's AI proxy) to generate coaching plans. There is no build, no tests and no framework: JavaScript (V8), HTML and CSS. The only external libraries are Chart.js (charts) and html2canvas (copy as image), both loaded from cdnjs (CDN).

The user speaks **Spanish**. Reply in Spanish, with simple explanations and examples. Code, comments and project docs are in English.

## Working rules (mandatory)

- **Work only on the COPY** of the project. Never on the original.
- **Ask before every `clasp push`, `git commit` and `git push`.** An approval covers that one action, not the next. When asking, say which files changed.
- Before a push, check that `scriptId` in `.clasp.json` is the copy's.
- **Do not use `clasp push --force`.** If clasp says "Skipping push", it is almost surely because the remote `appsscript.json` differs from the local one. Compare and report; do not overwrite.
- Do not propose `clasp push` or test code when the user only wants to verify something: they prefer UI walkthroughs.
- Do not implement improvements nobody asked for. Note them as suggestions.
- Never read, print or copy keys or tokens (`FUELIX_API_KEY`, `~/.clasprc.json`, `~/.claude/settings.json`). Do not paste them into files or the chat.
- Do not run local tests on your own: validation happens on the deployed copy.
- **Never commit** `.clasprc.json` or any `*.xlsx` / `*.csv` with real agent data. It contains personal information and the account is confidential; anonymize it first (same structure, fake data).

## clasp

- **Node 20 is required:** run `nvm use 20` in every new terminal. The default Node (v14) fails with clasp.
- clasp 3.x was installed from the public registry: `npm install -g @google/clasp --registry=https://registry.npmjs.org/` (TELUS's private CodeArtifact is no longer available).
- `clasp login` expires under TELUS's reauthentication policy: `invalid_grant … invalid_rapt`. Fix it with `nvm use 20 && clasp login` (it opens the browser; the user does it, with `! clasp login`).
- Normal flow: `clasp pull` → edit → `clasp status` → `clasp push`.
- `clasp push` **replaces** what is in Google with what is local, and deletes in the cloud any file that no longer exists here. Run `clasp pull` first if someone may have edited on the web.
- Local `.js` files are uploaded as `.gs`.
- A push does **not** touch Script Properties, triggers or Sheet data.
- `clasp pull` and `clasp push` move code only, never sheet data.
- `clasp` only tracks the extensions in `.clasp.json` (`.js`, `.gs`, `.html`, `.json`); `.md` files and `docs/` show as untracked and are not uploaded.
- To see changes: the `/dev` test URL (Deploy → Test deployments, always the latest code) or publish a new version.

### Deployments

- The `/exec` URL stays **pinned to one version**. To update it without changing the URL: Deploy → Manage deployments → pencil → *New version* → Deploy.
- *New deployment* creates a different URL and the old one stops serving.
- `clasp deployments` lists only active deployments, not archived ones.
- The remote manifest can differ from the local one, so compare before pushing. As of 2026-10-16 both the cloud and the local `appsscript.json` have `webapp.access = "DOMAIN"` (any TELUS account with the URL) and `executeAs = "USER_DEPLOYING"`. Changing `access` changes who can open the app, so never edit it as a side effect.
- To compare cloud against local safely: `clasp clone <scriptId>` into a **new, empty** scratchpad folder and `diff -rq`.

## Fuelix (AI)

- Used only by the Agent Dashboard (`FuelixClient.js`). The TL, OM and AGM dashboards do not call the AI, and must not unless the user asks.
- **The key lives in Script Properties** (Project Settings → Script Properties) under the exact name `FUELIX_API_KEY`. It is read by `getFuelixApiKey_()` (`Config.js`) on every call. It never goes in code.
- It is created in the Fuelix developer portal (`dev.fuelix.ai`). A **project-specific** key is recommended (revocable, with permission only for Chat Completions), not a personal one.
- Changing the key **does not require** a new deployment.
- **Other Script Property:** `HELP_LINK` (optional) holds the documentation URL for the "?" help button. `getHelpLink_()` (`Config.js`) only accepts `http(s)://` values (it blocks `javascript:`); if it is missing or invalid the button stays hidden.
- Call: `POST https://api.fuelix.ai/v1/chat/completions` with `Authorization: Bearer <key>`. Parameters are in `Config.js` (`FUELIX_API_BASE`, `FUELIX_MODEL`) and `FuelixClient.js` (`temperature` 0.4, `max_tokens` 4000).
- The response is expected to be JSON (markdown fences are stripped). If it is not 2xx, `Fuelix API error <code>: …` is thrown.
- The model in `CONFIG.FUELIX_MODEL` (`claude-sonnet-5`) has produced valid coaching plans (they are stored in `Processed_Metrics_Cache`), so the name works with the proxy as configured.
- Each agent's data (name, metrics, QA notes) leaves for an external API. Confirm the use is approved before extending it.
- AI consumption is not recorded anywhere in the app.

## Code structure and conventions

- A single shared global scope: no `import`/`require`. Private functions carry a `_` suffix.
- Google environment: no `fetch`, `window` or `console.log`. Use `SpreadsheetApp`, `HtmlService`, `UrlFetchApp`, `PropertiesService` and `Logger`.
- JS inside the `.html` files runs in the browser and calls the server with `google.script.run`.
- **Only one `google.script.run` call at a time**: several simultaneous ones can return a null payload. Chain them (list, then template, then data).
- Every payload must leave the server as plain JSON (`JSON.parse(JSON.stringify(...))`): `Date` objects break the serializer.
- Numbers are computed in `Metrics.js`, never by the AI or in the template.
- Every dashboard follows the same pattern: a `build…Payload` function on the server, a `client…` function in `Code.js`, an HTML template with `__DATA_JSON_PLACEHOLDER__` and a section in `Index.html`. For a new screen, copy the OM or AGM pattern.
- Code comments are in English. Keep it that way.

## Copy as image and the shared UI kit

- **Library:** html2canvas 1.4.1 from cdnjs, loaded in `Index.html` and in `AgmDashboardTemplate.html`. There is no build, so it is a script tag.
- **Framed sections.** A block wrapped in the CSS class `frame` (rounded, bordered) or `copyable` (an existing `.card` that keeps its own style) gets a "Copy section" button in its top-right corner. To make a new section copyable, add one of those classes; nothing else.
- **Where the buttons come from.** The AGM template carries its own CSS and JS (`addCopyButtons`, `copyFrameAsImage`). The Agent, TL and OM templates get theirs injected by `Index.html` when each iframe finishes loading (`setupFrameKit`, `FRAME_KIT_CSS`); their templates only mark sections with `frame` / `copyable`.
- **Whole screen.** The "Copy screen as image" button in the side menu (`copyScreen`, `Index.html`) renders the current view without the side menu: the `.wrap` (or `body`) of the active iframe, or `#view-upload` for Files settings. It expands scroll areas first so nothing is cut, and caps the image height (about 16,000 px).
- **Excluding things from the image:** put `data-html2canvas-ignore="true"` on the element (already used on the copy and Clean buttons, "Export as HTML" and "+ Add Session").
- **Clipboard.** `navigator.clipboard.write` with a `ClipboardItem` that receives the render promise. If the browser or the Apps Script frame blocks it, the PNG is downloaded instead and a message says so. The AGM iframe has `allow="clipboard-write"`; whether Google's outer frame allows the clipboard has not been confirmed.
- Tall tables are copied in full (for example the whole QA Call Log), not only the visible rows.

## Data and sheets

- The app reads **by tab name**; the names are in `Config.js` and are case-sensitive.
- `Roster` is **replaced entirely** on every QA upload. That is why `TL_Roster`, `OM_Roster` and `AGM_Roster` are separate, hand-maintained tabs.
- Joins are by **email, case-insensitive**. Emails in the leadership tabs must match the Roster's exactly.
- Leadership tabs: row 1 numbering, row 2 headers, data from row 3. If a tab is missing, the code returns `[]` (it does not fail).
- `OM_Roster` uses `Active`; `TL_Roster` and `Roster` use `Active?`. The code accepts both.
- Every history row has a `WeekKey` (the Monday, `yyyy-MM-dd`).
- A real-data version containing personal information (agent names and emails) exists outside this repo. **Do not copy it here** without anonymizing it.

## Known gotchas

1. **Duplicates on upload.** `appendToHistory_` only appends and does not deduplicate. Uploading the same file twice duplicates the rows (percentages almost the same, counts inflated). `History_Survey` is currently tripled.
2. **Surveys with the wrong `WeekKey`.** In `History_Survey` every row ended up with `2026-07-13` even though `Call Date` runs from 7/13 to 8/17. This is the batch-week fallback (`UploadHandler.js`, near line 81). The exact reason `Call Date (…)` is not being read has not been determined. Effect: the AGM by week shows "No data" for satisfaction.
3. **Weekly caches are permanent.** If a rule or constant changes, delete the affected rows from `Processed_Metrics_Cache` and `TL_Metrics_Cache`. Correcting a past week is not supported.
4. **The cache stores AI errors** (`DashboardServer.js`, where the payload is saved). A failure stays repeated even after the cause is fixed. When the Key Insight failed, the Agent Dashboard shows a **Clean** button: it only deletes that agent's cached row for that period (`clientCleanAgentCache` → `clearProcessedMetricsEntry`) and tells the user to reopen the view. It does **not** call the AI itself; the AI is called again when the agent is opened. The server refuses to delete a row whose plan is not an error. Other agents or periods with a stored error need their own Clean (or delete the row in `Processed_Metrics_Cache`). An upload also clears every `all` row.
5. **The UI hides the cause of an AI failure.** The text is always "Coaching insight unavailable.". The real error is in `payload.ascend.error`: in the Sheet, search for `Fuelix API error` (or `Fuelix API key not set`) in `Processed_Metrics_Cache`.
6. **`History_WFM` probably has a damaged header** (`WFM_UPLOAD_HEADER_ROW: 19`). AHT and occupancy may come out empty. That is why the AGM view has no Efficiency section.
7. **If a source is missing the expected tab**, `readSheetValues_` reads the first tab without warning.
8. **Survey or WFM rows the Roster does not recognize** are saved with an empty `EID` and no dashboard shows them. Upload QA first.
9. **No role-based access control.** `access` is `DOMAIN` or `MYSELF` depending on the deployment and the user is never identified: whoever opens the URL sees everything, including the upload screen. The "downward" rule (TL sees agents, OM sees TLs, AGM sees OMs) is **not implemented**. If it is implemented, also check it in every server-side `client*` function; hiding buttons protects nothing.
10. **`executeAs: USER_DEPLOYING`**: the app runs as the account that deploys it. If that account loses access, the app can stop working. That account also needs read access to the 3 sources.
11. The **Drive API** advanced service is enabled in the manifest but the code does not use it. The header comments of `UploadHandler.js` mention an `.xlsx` conversion that no longer exists: it now reads with `openById`.
12. `testDianaPayload` and `testTeamCachePayload` have a real agent hardcoded. Do not use them as an example.
13. A payload is cached in **one cell** (limit of about 50,000 characters). Watch it when adding data to a view.
14. **The AI response can arrive truncated.** It happened once (`Could not parse Ascend JSON output: Unterminated string in JSON`, response cut at about 4,200 characters). The cause was not determined. The error is cached like any other; use Clean and reopen the agent.
15. **`doGet` injects the `eid` URL parameter without escaping** into a `<script>` in `Index.html` (`__DEEP_EID_PLACEHOLDER__`). It is an injection risk and is not fixed. The help link, in contrast, is escaped.
