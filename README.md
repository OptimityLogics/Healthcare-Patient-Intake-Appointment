# Healthcare Patient Intake & Appointment Agent

An n8n workflow that provides a Telegram-based virtual clinic front desk for administrative patient intake and appointment scheduling. It uses an AI agent to collect patient details, check available appointment slots in Google Sheets, book a selected slot, and save appointment details.

> **Important:** This is an automation demo, not a medical device or a substitute for clinical care. The urgent-message check relies on keyword matching and may miss emergencies. It must not be relied on as the only emergency detection or escalation mechanism.

## Features

- **Telegram patient intake:** Receives patient messages and replies in the Telegram conversation.
- **Urgent-message routing:** Checks incoming text for a set of urgent keywords. Matching messages are sent to a staff Telegram chat and receive an emergency guidance reply; the normal booking-agent path is bypassed.
- **AI-assisted intake:** Uses Google Gemini through n8n's AI Agent to collect the patient's full name, age, Indian mobile number, and reason for visit.
- **Department routing:** Selects an administrative department based on the patient's stated reason, using the configured list: General Medicine, Orthopedics, Dermatology, Pediatrics, ENT, and Gynecology.
- **Availability lookup:** Reads rows from the Google Sheets `Slots` sheet where the department matches and status is `Available`.
- **Appointment booking:** Updates the selected slot's status to `Booked` and appends the appointment details to the `Appointments` sheet after the patient confirms a slot.
- **Human escalation:** Provides a tool for sending a summary to clinic staff when a human is needed or a tool fails.
- **Conversation context:** Uses a short window of conversation memory keyed by Telegram chat ID.

## Workflow Overview

1. A patient sends a Telegram message.
2. The workflow extracts the chat ID, message text, and first name, then performs a keyword-based urgent-message check.
3. If the message matches the urgent keywords, the workflow alerts the configured staff Telegram chat and sends an emergency response to the patient. It does not continue into the appointment agent on that branch.
4. Otherwise, the AI agent handles administrative intake and scheduling, using the connected Google Sheets and staff-escalation tools as needed.
5. After explicit patient confirmation, the agent is instructed to mark the slot as booked and save the appointment record.
6. The agent's response is sent back to the patient on Telegram.

## Repository Structure

```text
Healthcare-Patient-Intake-Appointment/
├── README.md
├── documentation/
│   └── OVERVIEW.md
├── setup/
│   └── SETUP.md
├── workflow/
    └── Healthcare_Patient_Intake_Appointment_Agent.json
```

## Requirements

- An n8n instance with the required Telegram, Google Gemini, and Google Sheets integrations/nodes available.
- A Telegram bot and configured Telegram credentials in n8n.
- A Google Gemini API credential configured in n8n.
- A Google Sheets document with two sheets:
  - `Slots`: expected columns include `Slot_ID`, `Department`, `Doctor`, `Date`, `Time`, and `Status`.
  - `Appointments`: expected columns include `Timestamp`, `Telegram_Chat_ID`, `Patient_Name`, `Age`, `Phone`, `Department`, `Reason_For_Visit`, `Slot_ID`, `Doctor`, `Date`, `Time`, and `Status`.
- A staff Telegram chat ID for urgent alerts and human escalation.

## Setup

1. Open the workflow JSON in the [workflow folder](workflow/Healthcare_Patient_Intake_Appointment_Agent.json) and import it into n8n.
2. Reconnect the Telegram and Google Gemini credentials in the relevant nodes.
3. Replace `YOUR_GOOGLE_SHEETS_DOCUMENT_ID` with the ID of your own Google Sheets document in each Google Sheets node.
4. Replace `YOUR_STAFF_TELEGRAM_CHAT_ID` with the intended staff Telegram chat ID in both staff notification nodes.
5. Confirm that the Google Sheets tab names and column headers match the workflow configuration. Add only test appointment slots and synthetic patient records during development.
6. Test normal intake, invalid phone numbers, no available slots, confirmed booking, tool failures, human escalation, and urgent-message routing using synthetic data.
7. Review the workflow's permissions and behavior, then activate it in n8n only when you are ready to test it in a controlled environment.

See [setup/SETUP.md](setup/SETUP.md) for additional setup guidance.

## Configuration Notes

- The workflow file intentionally does **not** contain your original Google Sheets document ID, staff Telegram chat ID, n8n credential IDs, webhook IDs, or instance ID.
- The JSON contains placeholders for the Google Sheets document ID and staff Telegram chat ID. Replace them locally in n8n; do not commit private identifiers or credentials to a public repository.
- The Telegram trigger and reply nodes require valid Telegram credentials. The Gemini model node also requires a working Google Gemini credential.
- The urgent keyword list is a basic text-matching safeguard, not a clinically validated triage system. It may produce false positives or miss urgent situations, including messages in languages or phrasing not covered by the configured patterns.
- The AI agent's instructions do not guarantee that the model will always follow every step. Verify the booking and appointment-save results in testing before relying on the workflow.

## Privacy, Security, and Safety

- Do not commit API keys, access tokens, credentials, real patient names, phone numbers, chat IDs, or medical information.
- Telegram messages and appointment details are processed by external services and Google Sheets. Before using real patient data, assess consent, access controls, retention, provider terms, and the healthcare privacy laws applicable to your location.
- Restrict access to the Google Sheets document and staff Telegram chat. Use least-privilege credentials and avoid sharing production data in logs or screenshots.
- This workflow is for administrative intake and scheduling only. It must not diagnose conditions, recommend medication, or provide treatment advice.
- In an emergency, patients should contact local emergency services or go to an emergency department. Do not depend on this bot to detect or respond to every emergency.

## Current Status

The sanitized n8n workflow JSON is included in this repository. Credentials, Google Sheets document ID, staff Telegram chat ID, and end-to-end behavior must be configured and verified in your own n8n environment before use.
