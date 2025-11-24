# Architecture Overview

```text
[Trigger]  -->  [Make Webhook]
                  |
                  v
             Tools: Set BASE_CCY
                  |
                  v
            HTTP: Fetch FX rates
                  |
                  v
         Tools: Store FX_RATES JSON
                  |
                  v
 Google Sheets: Get Range Values (actuals)
                  |
                  v
       JSON: Create JSON (Actuals_Structure)
                  |
                  v
  OpenAI: Generate MD&A commentary
                  |
                  v
 Notion: Create Run Log row
                  |
                  v
 Slack: Send summary message (optional)
```

- System boundary: everything runs inside a single Make scenario.
- Source systems: Google Sheets (Board_Model), FX API.
- System of record for runs: Notion database "Board Pack Run Log".
- Notification channel: Slack (optional).

See `workflow.json` for a high‑level tooling list.
