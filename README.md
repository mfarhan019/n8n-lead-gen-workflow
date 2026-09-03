# 🧲 Lead Generation Automation — n8n Workflow

An intelligent, fully automated lead generation pipeline built with **n8n**. This workflow captures incoming leads via a webhook, validates them in real-time, logs them to Google Sheets, and sends branded email notifications — all without any manual intervention.

---

## 📸 Workflow Screenshot

![Workflow Screenshot](screenshot.png)

## 🔄 Workflow Overview

> **Webhook → Field Extraction → Validation → Google Sheets → Admin Email → Client Confirmation**

```
Webhook (POST)
    └── Get Required Fields
            └── Validate Lead (Genuine / Fake)
                    └── Append to Google Sheets
                            └── Notify Admin via Email
                                    └── [IF Genuine] → Send Confirmation to Client
                                    └── [IF Fake]    → Skip (No Email Sent)
```

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔗 **Webhook Trigger** | Accepts `POST` requests from any form or frontend |
| 🧹 **Field Extraction** | Cleanly extracts `name`, `email`, `phone`, `treatmentType`, `date` |
| 🤖 **Smart Validation** | Detects fake emails (test/asdf/xyz patterns, repeated chars) and invalid phone numbers (sequential, repeated digits) |
| 📊 **Google Sheets Logging** | Every lead (genuine or fake) is logged with its status |
| 📧 **Admin Alert Email** | Sends a styled HTML email to the admin for every lead |
| ✅ **Client Confirmation Email** | Sends a branded thank-you email — only to genuine leads |
| 🚫 **Fake Lead Suppression** | Fake leads are logged but never emailed |

---

## 🔄 Workflow Step-by-Step

### 1. `Webhook`
- **Method:** `POST`
- **Path:** `/lead-generation`
- Receives raw lead data from your contact form or frontend

### 2. `Get Required Fields Only`
- Extracts only the needed fields from the raw webhook body
- Formats the date as `yyyy-MM-dd` using `$now`

### 3. `Checks if the incoming lead is genuine or fake`
- **Name:** Converts to Title Case (e.g. `"john doe"` → `"John Doe"`)
- **Email checks:**
  - Must match a valid email regex
  - Must not contain patterns like `test`, `fake`, `example`, `asdf`, `xyz`
  - Local part must not be all repeated characters
- **Phone checks:**
  - Must be 10–15 digits (with optional `+` prefix)
  - Must not be all the same digit (`1111111111`)
  - Must not be sequential (`1234567890`, `0987654321`)
- Sets `status` to either `"Genuine Lead"` or `"Fake Lead"`

### 4. `Append new lead in sheet`
- Appends all leads (genuine + fake) to Google Sheets
- Columns: `Name`, `Email`, `Phone No.`, `Treatment`, `Date`, `Status`

### 5. `Send a message to ME`
- Sends a styled HTML notification email to the admin
- Subject includes lead name and status
- Status badge is **green** for genuine, **red** for fake

### 6. `Send Email only if it is a Genuine Lead` (IF Node)
- Routes to client email if `status === "Genuine Lead"`
- Routes to a no-op node if the lead is fake

### 7. `Send a message to CLIENT`
- Sends a branded confirmation email to the lead's email address
- Mentions the requested treatment and a 24-hour response promise

---

## 🛠️ Setup Instructions

### Prerequisites
- [n8n](https://n8n.io/) (self-hosted or cloud)
- Gmail account with OAuth2 configured in n8n
- Google Sheets account with OAuth2 configured in n8n
- A Google Sheet with the following columns in `Sheet1`:

  | Name | Email | Phone No. | Teatment | Date | Status |
  |------|-------|-----------|----------|------|--------|

### Step 1 — Import the Workflow
1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Upload `workflow.json`

### Step 2 — Configure Credentials
Update the following nodes with your own credentials:

| Node | Credential Needed |
|---|---|
| `Append new lead in sheet` | Google Sheets OAuth2 |
| `Send a message to ME` | Gmail OAuth2 |
| `Send a message to CLIENT` | Gmail OAuth2 |

### Step 3 — Update Placeholders

Search for and replace the following values inside the workflow or via the n8n UI:

| Placeholder | Replace With |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Your actual Google Sheets document ID |
| `YOUR_ADMIN_EMAIL@gmail.com` | Your admin notification email address |
| `Your Clinic Name` | Your business/clinic name |
| `YOUR_GMAIL_CREDENTIAL_ID` | Your n8n Gmail credential ID |
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | Your n8n Google Sheets credential ID |

### Step 4 — Activate the Workflow
1. Click **Activate** (toggle in the top right)
2. Copy the webhook URL from the **Webhook** node
3. Use it as the `action` endpoint in your contact form

---

## 📬 Sample Webhook Payload

Send a `POST` request to your webhook URL with this JSON body:

```json
{
  "name": "Sarah Johnson",
  "email": "sarah.johnson@example.com",
  "phone": "+923001234567",
  "treatmentType": "Laser Hair Removal"
}
```

---

## 📁 Project Structure

```
lead-gen-n8n/
├── workflow.json       # Main n8n workflow (import this)
├── screenshot.png      # Workflow canvas screenshot
├── README.md           # This file
└── .gitignore          # Git ignore rules
```

---

## ⚠️ Security Notes

- **Never commit real credentials** — the workflow file has been sanitized with placeholder values
- **Credential IDs** in the JSON are for reference only; n8n will re-map them on import
- Consider adding **IP whitelisting** or a **secret header** to the webhook for production use

---

## 🧩 Customization Ideas

- Add a **Slack / Discord notification** node alongside the admin email
- Connect **Twilio** for SMS alerts on new genuine leads
- Add **CRM integration** (HubSpot, Notion, Airtable) after Google Sheets
- Extend validation with an **email existence checker** API
- Add **rate limiting** logic to prevent webhook flooding

---

*Built with ❤️ using [n8n](https://n8n.io/)*
