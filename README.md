# Holiday Email Atelier Workflow (n8n + LLM)

## Overview

This project contains an n8n workflow and a small HTML/JS frontend that let you:

- Upload or reference a contacts list (CSV or Google Sheet)
- Configure campaign settings (tone, sender, offer, schedule)
- Generate personalized holiday email content via an LLM agent
- Optionally update your data source with send status

All environment-specific URLs, API keys, and credentials are replaced with placeholders so you can safely adapt it to your own stack.

## Features

- Web UI (“Seasonal Email Atelier”) for non-technical users
- CSV upload with:
  - Email validation
  - Deduplication by email
  - Quick preview of first few contacts
- Google Sheets support by URL
- Configurable:
  - Campaign name
  - Sender name
  - Tone (warm, festive, professional, casual)
  - Optional holiday offer text
  - Optional scheduled send time
- Backend workflow to:
  - Receive UI submissions via webhook
  - Normalize contacts
  - Call an LLM agent API to generate subject + HTML body
  - Loop over contacts and send personalized messages
  - Optionally write back to a sheet with status

## Architecture

1. **Frontend (HTML/JS)**  
   - Renders the holiday studio UI.  
   - Handles CSV parsing, deduping, and previewing contacts in the browser.  
   - Builds a JSON payload with:
     - `dataSource`: `"csv"` or `"sheets"`
     - `contacts`: array (for CSV)
     - `sheetUrl`: URL (for Sheets)
     - `campaignName`, `tone`, `senderName`, `offer`, `scheduleTime`
     - LLM agent chat ID
   - Sends payload to the backend webhook.

2. **Webhook entrypoint**  
   - Receives the JSON payload from the UI.  
   - Branches based on `dataSource`:
     - CSV: uses provided contacts directly.
     - Sheets: reads rows from the specified sheet URL.

3. **Contacts normalization**  
   - Validates and normalizes:
     - `name`
     - `email`
     - `company`
     - `purchase_history`
   - Filters invalid or duplicate emails.

4. **LLM agent calls**  
   - For each contact, sends:
     - Name, company, purchase history
     - Tone, sender name, optional offer  
   - Receives structured JSON with `subject` and `html_body`.

5. **Email sending / sheet updating**  
   - Uses the generated content to send emails via your chosen mail provider.  
   - Optionally writes back send status into the source (e.g., a “status” column in Sheets).

## Setup

### 1. Import the workflow

1. Open your n8n instance.
2. Create a new workflow.
3. Use **Import from file** and select the json file.
4. Save the workflow.

### 2. Configure credentials

In n8n, create the following credentials and assign them to the relevant nodes:

- **Google Sheets OAuth2**  
  - Access to the sheet you plan to use.
- **Email / SMTP (or other email integration)**  
  - Used by the email-sending node(s).
- **LLM agent / HTTP API**  
  - Create an HTTP credential or store API keys in environment variables.

Replace any placeholder IDs or credential references in the imported nodes with your own credentials.

### 3. Replace placeholders

Search in the workflow (and UI HTML/JS if you host it separately) and replace:

- `YOUR_TOOLHOUSE_API_KEY`  
  → Your real LLM/agent API key.
- `YOUR_TOOLHOUSE_CHAT_ID`  
  → Your LLM agent / chat configuration ID.
- `https://your-n8n-instance.com/webhook/holiday-submit`  
  → The actual webhook URL for your entry node.
- `YourCompany`  
  → Your brand or company name.
- `your-email@example.com` / `recipient@example.com`  
  → Default sender or test addresses as needed.
- `YOUR_GOOGLE_CREDENTIAL_ID`, `YOUR_SMTP_CREDENTIAL_ID`  
  → With the actual credential IDs in the node configs.

Ensure this information stays in environment variables or n8n credentials in production.

### 4. Host the UI (optional)

The UI HTML is embedded as a response in one of the webhook nodes. You have two options:

- **Use it directly from n8n**  
  - Call the corresponding “UI” webhook URL in your browser.  
  - n8n will serve the holiday studio page.

- **Host it yourself**  
  - Copy the HTML/JS out to your own static hosting.  
  - Point it at your n8n webhook URL in the `CONFIG` object:

