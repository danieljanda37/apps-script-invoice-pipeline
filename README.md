# Email Approval Pipeline · Google Apps Script

End-to-end invoice (or any document) approval workflow built entirely on Google Workspace — Gmail, Drive, Sheets — with zero external infrastructure.

> **Note**: This is a **generic, anonymized** version of a system in production at SkiResort Černá hora · Pec, where it replaces paper-based invoice approvals across 6 ski areas. Names, IDs, and business specifics are placeholders — adapt to your context.

---

## What it does

```
            ┌────────────┐
            │  Supplier  │
            └─────┬──────┘
                  │ sends invoice email
                  ▼
        ┌──────────────────┐
        │  central inbox   │ ← Gmail filter routes by recipient
        └────────┬─────────┘
                 │
       ┌─────────┴──────────┐
       ▼                    ▼
┌──────────────┐    ┌──────────────┐
│ assignment   │    │ direct route │
│  Add-on      │    │ (e.g. gastro)│
│ (manual)     │    │              │
└──────┬───────┘    └──────┬───────┘
       │                   │
       ▼                   ▼
┌──────────────────────────────────┐
│  manager approval Add-on (1st)   │ ← in their own inbox
│  → choose cost center, note      │
│  → forward back, label "Stage 1" │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  director approval Add-on (2nd)  │ ← optional second stage
│  → final note, digital signature │
│  → forward back, label "Stage 2" │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  scheduled processor (every 15m) │
│  → pull labeled threads          │
│  → save PDFs to Drive            │
│  → write metadata to Sheet       │
└──────────────────────────────────┘
```

End result: every approved invoice is auto-filed in Google Drive with metadata logged in a single Google Sheet, ready for accounting.

---

## Why Apps Script

Most "low-code" automation platforms (Make.com, Zapier, n8n) charge per execution. For a workflow processing 100+ invoices/month with multiple stages each, that adds up. Apps Script is:

- **Free** for a Workspace account
- **Native** to Gmail/Drive/Sheets — no auth tokens to manage
- **Serverless** — scheduled triggers built in
- **Audit-friendly** — every execution logged in the script's stackdriver

Trade-off: no fancy UI builder. You write JavaScript. For a developer comfortable with code, that's a feature, not a bug.

---

## Repository structure

```
apps-script-invoice-pipeline/
├── README.md                     ← you are here
├── LICENSE                       ← MIT
├── docs/
│   ├── architecture.md           ← system design, data flow
│   ├── setup-guide.md            ← step-by-step deployment
│   └── adapting.md               ← how to adapt for your business
├── 01-stage-one-approval/        ← Gmail Add-on for first approver
│   ├── appsscript.json
│   ├── Code.gs
│   └── config.example.gs
├── 02-stage-two-approval/        ← Gmail Add-on for second approver
│   ├── appsscript.json
│   ├── Code.gs
│   └── config.example.gs
├── 03-single-stage-approval/     ← simplified version (one approver)
│   ├── appsscript.json
│   ├── Code.gs
│   └── config.example.gs
├── 04-leader-assignment/         ← Add-on to forward to specific manager
│   ├── appsscript.json
│   ├── Code.gs
│   └── config.example.gs
└── 05-invoice-tracker/           ← Time-triggered processor → Drive + Sheet
    ├── appsscript.json
    ├── Code.gs
    └── config.example.gs
```

Each project is **standalone** — deploy only what you need. The simplest setup uses just `03-single-stage-approval` + `05-invoice-tracker`.

---

## Quick start

1. **Prepare a Google Sheet** with two tabs:
   - `Invoices` — columns: Sender, Subject, PDF Link 1, PDF Link 2, Received, Cost Center, Note, Stage 1 Time, Stage 2 Status, Stage 2 Time, Processed, Message ID
   - `Cost Centers` — list of your departments / projects

2. **Prepare a Google Drive folder** for PDF storage. Copy its ID from the URL.

3. **Choose your flow:**
   - One approver → `03-single-stage-approval` + `05-invoice-tracker`
   - Two approvers → `01-stage-one-approval` + `02-stage-two-approval` + `05-invoice-tracker`
   - Mixed (different routes) → all four

4. **For each project you choose:**
   - Go to [script.google.com](https://script.google.com) → New project
   - Paste contents of `Code.gs` and replace `appsscript.json` (you need to enable manifest editing in Project Settings)
   - Copy `config.example.gs` to `config.gs` and fill in your IDs / emails / labels
   - Deploy as Gmail Add-on (Deploy → Test deployments → Install)

5. **Set up Gmail filters** (see `docs/setup-guide.md`)

6. **Set up time trigger** for `05-invoice-tracker`:
   - In its Apps Script editor → Triggers (clock icon) → Add Trigger
   - Function: `runAllProcessors`
   - Event source: Time-driven
   - Type: Minutes timer → Every 15 minutes

Detailed walkthrough in [`docs/setup-guide.md`](docs/setup-guide.md).

---

## Adapting to your business

This is built for invoices but the pattern works for any **email-driven approval workflow**:

- Travel expense reports
- Time-off requests
- Purchase orders
- Vendor onboarding paperwork
- Contract reviews

See [`docs/adapting.md`](docs/adapting.md) for guidance on adjusting:
- Number of approval stages
- Cost center / category lists
- Drive folder structure
- Sheet schema
- Label naming
- Email templates

---

## What's NOT included

- **Auth / SSO** — relies on Google Workspace login
- **Reporting dashboard** — Sheets is your dashboard. For BI, export to BigQuery (separate concern).
- **Mobile UI** — works in Gmail mobile app, but it's a Workspace Add-on. No standalone mobile.
- **Retry logic for failed forwards** — Apps Script has built-in retries, but for high-stakes workflows add explicit error logging.

---

## Built by

[Daniel Janda](https://danieljanda.cz) — IT & Systems Manager at SkiResort Černá hora · Pec.

Built originally to digitize invoice approvals across 6 ski areas. Now generic enough to share.

---

## License

MIT — see [LICENSE](LICENSE).

Use freely. Attribution appreciated but not required.
