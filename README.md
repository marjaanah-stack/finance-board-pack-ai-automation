# Finance Board Pack AI Automation

[![Status](https://img.shields.io/badge/status-prototype-blue.svg)](#)
[![Automation](https://img.shields.io/badge/stack-Make%2C%20OpenAI%2C%20Notion%2C%20Slack-green.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

AI-assisted board-pack generator for a multi-entity finance setup.
It pulls actuals from Google Sheets, fetches FX rates, converts the
sheet into machine-readable JSON, asks OpenAI to draft MD&A-style
commentary, and logs everything into Notion (with optional Slack
notification).

---

## 1. Repository structure

1. Repository structure

```text
board_pack_automation/
├─ README.md
├─ LICENSE
├─ workflow.json           # High-level list of Make modules
├─ data/
│  └─ board_model_example.csv   # Example "Board_Model" Google Sheet
├─ docs/
│  └─ architecture.md      # Text architecture diagram
├─ demo_output/
│  ├─ sample_notion_row.md
│  └─ sample_slack_message.md
└─ screenshots/
   └─ .gitkeep             # Drop screenshots here
```

Once you add PNG/JPG files to `screenshots/`, they will render in this
README via the gallery below.

## 2. Screenshot gallery

```Key points:

- The repo tree is wrapped in a fenced code block (` ```text ... ``` `).
- The three `![...]()` image lines are **outside** that code block and start at column 1 (no leading spaces).

### 3. Commit

1. Scroll down, add a small commit message (e.g. `Fix screenshot rendering`).
2. Click **Commit changes**.

Reload the README; the three images should now render.  

```

## 3. How the Make scenario works (high level)

1. **Webhooks – Custom Webhook**  
   Entry point. Opening the webhook URL (or calling it from another
   tool) triggers the whole run.

2. **Tools – Set variable (BASE_CCY)**  
   Stores the base reporting currency (e.g. `GBP`).

3. **HTTP – Make a request (FX API)**  
   Calls your FX provider (e.g. exchangerate.host / ECB demo).  
   Response is mapped to a variable and later stored in Notion as raw JSON.

4. **Tools – Set variable (FX_RATES)**  
   Keeps the selected `data.rates` object handy for later use.

5. **Google Sheets – Get Range Values**  
   Reads `Board_Model` → `actuals!A1:E`  
   Headers: `entity, currency, period_yyyy_mm, metric, amount`.

6. **JSON – Create JSON (Actuals_Structure)**  
   Wraps the rows into a JSON array with a fixed schema:
   `entity, currency, period_yyyy_mm, metric, amount`.

7. **OpenAI – Generate a completion**  
   Model: `gpt-4o` (or later).  
   Prompt: describe the group performance based on the JSON rows, in
   professional MD&A language.

8. **Notion – Create a Database Item (Legacy)**  
   Database: `Board Pack Run Log`.  
   Fields typically mapped:
   - `Run ID` – static prefix + period (e.g. `2025-07-board-pack`).  
   - `Period` – from the sheet (e.g. `2025-07`).  
   - `Entities` – text list of entities in the run.  
   - `Base Currency` – from `BASE_CCY`.  
   - `FX Rates (JSON)` – raw JSON string from the HTTP step.  
   - `MD&A Summary (first 500 chars)` – trimmed completion text.

9. **Slack – Send a Message (optional)**  
   Posts a short summary plus a link to the Notion row / Sheets / drive.

See `docs/architecture.md` for an ASCII diagram.

## 4. How to reuse this for other companies

1. Duplicate the Google Sheet and change:
   - Entity names
   - Periods
   - Metrics / amounts

2. Point the Google Sheets module to the new spreadsheet ID but keep
   the same header structure.

3. Point the Notion module to a new “Board Pack Run Log” database
   for that client.

4. Optionally change the prompt in the OpenAI module to match the
   client’s tone / KPI focus (SaaS, infra, cleantech, etc.).

The rest of the scenario architecture remains identical, so you can scale
this across multiple startups with minimal changes.

## 5. Demo artefacts

- `data/board_model_example.csv` – ready to import into Google Sheets as
  your `Board_Model` → `actuals` tab.  
- `demo_output/sample_notion_row.md` – a typical Notion row.  
- `demo_output/sample_slack_message.md` – the Slack summary text.

## 6. License

This project is released under the [MIT License](LICENSE). Use it as a
template for your own finance-automation portfolio.
