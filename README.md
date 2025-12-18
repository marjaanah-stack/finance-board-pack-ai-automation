# Finance Board Pack AI Automation

This is an automation that turns monthly Google Sheets actuals into MD&A commentary (through OpenAI API), writes to Notion, and notifies Slack — triggered via Webhook.

---

## 1. Repository structure

```
finance-board-pack-ai-automation/
├─ README.md
├─ LICENSE
├─ workflow.json                 # High-level list of Make modules
├─ data/
│   └─ board_model_example.csv   # Example “Board_Model” Google Sheet
├─ demo_output/
│   ├─ sample_notion_row.md
│   └─ sample_slack_message.md
├─ docs/
│   └─ architecture.md           # Text architecture diagram
├─ screenshots/
│   ├─ Google_Sheets_Actuals.png
│   ├─ Make_Scenario_Overview.png
│   └─ Notion_DB_Structure.png
```

---

## 2. Screenshot gallery

Images are stored in `screenshots/` and linked relative to README.

### Google Sheet structure
![Google Sheets](screenshots/Google_Sheets_Actuals.png)

### Make.com automation scenario
![Make Scenario](screenshots/Make_Scenario_Overview.png)

### Notion database structure
![Notion DB](screenshots/Notion_DB_Structure.png)

---

## 3. How the Make scenario works (high level)

### 1. **Webhooks – Custom webhook**
Entry point. Opening the webhook URL (or calling it from another tool) triggers the full run.

### 2. **Tools – Set Variable (`BASE_CCY=…`)**
Stores the base reporting currency (e.g., `GBP`).

### 3. **HTTP – Make a request (FX API)**
Calls your FX provider (e.g., ExchangeRate.host / ECB demo).  
Response is mapped to a variable and later stored in Notion as raw JSON.

### 4. **Tools – Set variable (`FX_RATES`)**
Keeps the selected data rates object handy for later use.

### 5. **Google Sheets – Get Range Values**
Reads **Board_Model** → **‘actuals’ tab**  
Headers: `entity, currency, period_yyyy_mm, metric, amount`.

### 6. **JSON – Create JSON (`Actuals_Structure`)**
Wraps the rows into a JSON array with fixed schema:  
`entity, currency, period_yyyy_mm, metric, amount`.

### 7. **OpenAI – Generate a completion**
Model: `gpt-4o` (or later).  
Prompt: describes the group performance based on the JSON rows, in professional MD&A language.

### 8. **Notion – Create a Database Item (Legacy)**
Database: **Board Pack Run Log**  
Fields typically mapped:
- **Run ID** – timestamp or static prefix + period
- **Period** – from the sheet (e.g., `2025-07`)
- **Entities** – list of entities found
- **Base Currency** – from JSON
- **FX Rates (JSON)** – raw JSON string from HTTP step
- **MD&A Summary** – trimmed OpenAI completion text  
- **Board Pack PPTX Link** – optional future extension  
- **Requested By** – webhook source / caller

### 9. **Slack – Send a Message (optional)**
Posts a short summary plus a link to the Notion row / Sheets / Drive.

For a visual overview, see `docs/architecture.md`.

---

## 4. How to reuse this for other companies

### Step 1 – Duplicate items
Duplicate the:
- Google Sheet  
- Notion database  
- Periods / months  
- Metrics / amounts structure

### Step 2 – Point the Google Sheets module to the new spreadsheet  
Keep the same header structure.

### Step 3 – Point the Notion module to the new **Board Pack Run Log** database for that client.

### Step 4 – Modify the OpenAI prompt  
Optionally adapt to each client’s tone, KPIs, sector, metrics.

### Step 5 – Everything else stays the same  
The scenario architecture remains identical, enabling scalable reuse across your fractional-CFO portfolio.

---

## 5. Demo artefacts

Located inside `/data` and `/demo_output`:

- `data/board_model_example.csv` — ready to import into Google Sheets as your “Board_Model → actuals” tab  
- `demo_output/sample_notion_row.md` — typical Notion row output  
- `demo_output/sample_slack_message.md` — Slack summary preview

---

## 6. License

This project is released under the MIT License. Use it freely as a template for your finance-automation portfolio.

---
