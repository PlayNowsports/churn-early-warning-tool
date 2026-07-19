# Churn Early-Warning Assistant

A reusable, browser-based renewal risk tool. Upload a pipeline export, apply calibrated rules, get a ranked churn watch-list.

**Live demo:** _add your GitHub Pages URL here after step 5 below_

---

## What it does

- Accepts an .xlsx / .xls / .csv renewal pipeline export
- Applies **structured rules** (Health field, Utilization thresholds, Age, Stage) deterministically in-browser
- Applies **unstructured rules** to CSM Notes and Gong Highlights using the Anthropic API (your key, batched calls)
- Produces a ranked risk list with per-account reasoning (which rules fired, why)
- Exports results to CSV

## What makes it reusable

The core value isn't the tool logic — it's the **rule set**. Rules are fully CRUD-managed in-browser, persisted in `localStorage`, and exportable as JSON so they can be versioned in git or shared across an analyst team. Ships with 12 calibrated defaults; edit, replace, or extend them.

| Feature | Behavior |
|---|---|
| **Create** rule | Modal form with type-aware fields, validation on save |
| **Read** rules | Sortable table with active/inactive toggle, condition preview, reasoning |
| **Update** rule | Same modal; unique-key check for AI pattern keys |
| **Delete** rule | Confirmation prompt; no soft-delete (use "inactive" for that) |
| **Persist** rules | `localStorage` under key `churn_tool_rules_v1` |
| **Portability** | Export / import as JSON; reset to defaults |

Column mapping and last-run results also persist, so you don't re-map the same file every session.

## How risk scoring works

Every active rule that matches an account contributes its score. Totals map to tiers:

| Tier | Score |
|---|---|
| Critical | 60+ |
| Elevated | 30 – 59 |
| Watch | 15 – 29 |
| Healthy | < 15 |

Structured rules are 100% deterministic. Only unstructured (AI) rules touch the API, and the batch prompt is versioned in the source so runs are reproducible at temperature 0.

## Privacy

Everything runs in your browser. The only network call is to `api.anthropic.com`, using your own API key, only when AI rules are active. Nothing is uploaded, logged, or stored on any server.

---

## Deploy to GitHub Pages (5 steps, ~2 minutes)

1. **Create a new GitHub repo** (public — Pages requires public on the free tier, or Pro for private Pages).

2. **Push these files:**
   ```bash
   git init
   git add index.html README.md
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```

3. In the GitHub repo, open **Settings → Pages**.

4. Under **Build and deployment**, set:
   - Source: **Deploy from a branch**
   - Branch: **main** · Folder: **/ (root)**
   - Click **Save**.

5. Wait ~30 seconds. GitHub gives you a URL like `https://<your-username>.github.io/<repo-name>/`. That's the link you send.

To update: edit `index.html`, commit, push. Pages redeploys automatically.

## Run locally

Just open `index.html` in a browser. No build step, no server, no dependencies to install. All libraries (SheetJS, fonts) load from CDN.

## File structure

```
tulip-churn-tool/
├── index.html      Single-file app (~35KB)
└── README.md       This file
```

## Notes for the AI Workflow Log

- Structured vs unstructured signal separation is enforced at the rule schema level — an AI-tagged risk can never silently override a deterministic Closed Lost.
- Pattern keys must be unique — the save handler blocks duplicates.
- Batch size (default 25) was chosen to bring a 700-row run from ~35 minutes (one call per account) to ~2 minutes (~28 calls). Configurable in the Run tab.
- API key is stored in session-scoped state only, never in `localStorage`.
- Rule schema is versioned (`_v1` suffix on storage keys) so future breaking changes can migrate without wiping user rules.

## License

MIT — do whatever you want with it.
