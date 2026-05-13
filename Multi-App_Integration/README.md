# Telegram → Google Sheets Lead Logger

Automation workflow built in n8n. Captures inbound Telegram bot messages and logs them as structured rows in a Google Sheet for real estate lead tracking.

## Repository Structure

```
/
├── lab_summary.md          ← real-world justification + technical write-up
├── workflow.json           ← n8n workflow export (import via n8n UI)
├── screenshots/
│   ├── 01_workflow_canvas.png     ← n8n canvas showing all 3 nodes connected
│   └── 02_google_sheets_rows.png  ← Google Sheet with populated lead rows
└── README.md               ← this file
```

## How to Run

### Prerequisites
- n8n account (cloud or self-hosted)
- Telegram account + bot token from @BotFather
- Google account with Google Sheets API and Google Drive API enabled
- Google Cloud OAuth2 credentials (Client ID + Client Secret)

### Steps

1. **Import workflow**
   - In n8n: Menu → Import from file → select `workflow.json`

2. **Configure Telegram credential**
   - Open Telegram Trigger node → Credential → Create new
   - Paste your bot token from @BotFather → Save

3. **Configure Google Sheets credential**
   - Open Google Sheets node → Credential → Create new → OAuth2
   - Paste Client ID and Client Secret from Google Cloud Console
   - Click Sign in with Google → allow access

4. **Create the Google Sheet**
   - Create a sheet named `Real Estate Lead Tracker`
   - Rename the tab to `Inbound Leads`
   - Add headers in row 1: `Timestamp`, `Sender Username`, `Sender Name`, `Chat ID`, `Message Text`, `Status`, `Source`

5. **Point the Google Sheets node to your sheet**
   - Document → From list → select `Real Estate Lead Tracker`
   - Sheet → select `Inbound Leads`

6. **Activate the workflow**
   - Click the Inactive toggle (top right) → turns green → Published

7. **Test**
   - Send a message to your Telegram bot
   - Check the `Inbound Leads` tab — a new row should appear within seconds

## Field Mapping

| Google Sheet Column | Source |
|---------------------|--------|
| Timestamp | `$now` |
| Sender Username | `$json.message.from.username` (fallback: `no_username`) |
| Sender Name | `$json.message.from.first_name + last_name` |
| Chat ID | `String($json.message.chat.id)` |
| Message Text | `$json.message.text` |
| Status | `New` (literal) |
| Source | `Telegram` (literal) |
