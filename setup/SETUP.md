# Setup Guide — Healthcare Patient Intake & Appointment Agent

This guide explains how to configure the n8n workflow in `workflow/Healthcare_Patient_Intake_Appointment_Agent.json`.

The workflow connects Telegram, Google Gemini, and Google Sheets to collect administrative patient-intake details, check appointment availability, book a selected slot, save appointment details, and escalate urgent or unclear cases to clinic staff.

> **Important:** This is an administrative automation example, not a medical triage system. The urgent-message check uses a limited list of keywords and can miss emergencies. Do not rely on it to determine whether a patient is safe. Have qualified staff review the workflow and its safeguards before any real-world use.

## 1. Prerequisites

Prepare the following before importing the workflow:

- An n8n instance with permission to create and run workflows.
- A Telegram bot created through Telegram's official BotFather and a Telegram credential configured in n8n.
- A Google Gemini API key and a Google Gemini credential configured in n8n.
- A Google account with access to the Google Sheets document used for appointment slots and records.
- A staff Telegram chat or group where urgent alerts and human-escalation messages can be received.
- The required n8n nodes/packages for Telegram, Google Sheets, and the LangChain AI Agent / Google Gemini Chat Model nodes. Confirm compatibility with your installed n8n version.

## 2. Import the workflow

1. Download `workflow/Healthcare_Patient_Intake_Appointment_Agent.json` from this repository.
2. Open your n8n workspace.
3. Use **Import from File** (the exact menu location can vary by n8n version) and select the JSON file.
4. Open the imported workflow and inspect all nodes for warnings or missing credentials.
5. Reconnect credentials as described below.
6. Configure the two placeholder values described in Section 4.
7. Save the workflow, then run the test checklist before enabling it for ongoing use.

The exported JSON contains placeholder values rather than the original account identifiers and credential references. Importing it does **not** connect your Telegram bot, Gemini account, or Google Sheet automatically.

## 3. Configure credentials

Create or select credentials in n8n and assign them to the matching nodes.

### Telegram

Connect your Telegram credential to these nodes:

- `Patient Message (Telegram)`
- `Alert Staff (Urgent)`
- `Urgent Reply to Patient`
- `Escalate to Human Staff`
- `Reply to Patient`

The bot must be able to receive patient messages and send replies. Add it to the staff group, if you use a group, and confirm it has permission to send messages there. Test that the configured staff chat ID is the intended destination before activating the workflow.

### Google Gemini

Connect your Google Gemini credential to `Gemini Chat Model`.

- Model configured in the JSON: `models/gemini-3.1-flash-lite`
- Ensure your API key and account have access to the selected model.
- Model availability and node options may vary by provider and n8n version.

### Google Sheets

Connect your Google Sheets OAuth2 credential to all three Google Sheets tool nodes:

- `Check Available Slots`
- `Book Slot`
- `Save Patient Appointment`

The connected Google account must have permission to read and update the `Slots` sheet and append rows to the `Appointments` sheet.

## 4. Set the required placeholders

Use the workflow editor to find and replace these placeholders in the relevant nodes.

| Placeholder | Where it is used | What to enter |
|---|---|---|
| `YOUR_GOOGLE_SHEETS_DOCUMENT_ID` | All three Google Sheets tool nodes | The ID of your Google Sheets document |
| `YOUR_STAFF_TELEGRAM_CHAT_ID` | `Alert Staff (Urgent)` and `Escalate to Human Staff` | The Telegram chat ID for the staff member or group that should receive alerts |

### Google Sheets document ID

Open your Google Sheets document and copy its document ID from the URL. In a URL such as `https://docs.google.com/spreadsheets/d/DOCUMENT_ID/edit`, the document ID is the part between `/d/` and `/edit`.

Replace the placeholder in **all three** Google Sheets nodes. Confirm the sheet tab names are exactly `Slots` and `Appointments`.

### Staff Telegram chat ID

Set the correct staff chat ID in both staff-notification nodes. Do not leave the placeholder in place. Keep the ID private if it identifies a private staff chat.

## 5. Prepare the Google Sheets document

The workflow expects one Google Sheets document containing two tabs with the following exact names and column headers. Keep the header spelling consistent with the workflow configuration.

### Tab 1: `Slots`

Create these columns in row 1:

| Column | Purpose | Example test value |
|---|---|---|
| `Slot_ID` | Unique identifier used to update a selected slot | `SLOT-001` |
| `Department` | Department name | `General Medicine` |
| `Doctor` | Doctor or provider name | `Dr. Example` |
| `Date` | Appointment date | `2026-11-10` |
| `Time` | Appointment time | `10:30 AM` |
| `Status` | Slot availability | `Available` |

Add test slots for the departments used by your clinic. The workflow filters availability by the exact `Department` value and `Status = Available`. The agent's configured department names are:

- General Medicine
- Orthopedics
- Dermatology
- Pediatrics
- ENT
- Gynecology

Use consistent department names and date/time formats. Each `Slot_ID` should identify one slot uniquely. A successful booking updates the matching row's `Status` to `Booked`.

### Tab 2: `Appointments`

Create these columns in row 1, in the same spelling:

`Timestamp`, `Telegram_Chat_ID`, `Patient_Name`, `Age`, `Phone`, `Department`, `Reason_For_Visit`, `Slot_ID`, `Doctor`, `Date`, `Time`, `Status`

After the booking tool succeeds, the workflow is configured to append an appointment row with the collected patient and slot details. The status written by the workflow is `Confirmed`.

Use synthetic data while testing. Treat this sheet as sensitive because it can contain personal and appointment information. Restrict access to only the people and service accounts that need it.

## 6. Understand the workflow paths

### Urgent-message path

The `Prepare Message + Urgent Check` node checks the incoming text against a fixed set of urgent keywords. If a match is found:

1. `Alert Staff (Urgent)` sends the patient's message and Telegram chat ID to the configured staff chat.
2. `Urgent Reply to Patient` advises the patient to contact emergency services or go to an emergency department. The message mentions 112 for India.

This branch does not book an appointment. The keyword list is only a basic safeguard: it can produce false positives and false negatives, and it does not replace clinical assessment or emergency procedures. Verify the alert reaches staff and establish a reliable human response process.

### Standard intake and booking path

If the message does not match the urgent keyword check, the workflow sends it to `Patient Intake Agent`. The configured agent is intended to:

1. Collect the patient's full name, age, 10-digit Indian mobile number, and reason for visit.
2. Select a department from the configured list.
3. Use `Check Available Slots` to find slots marked `Available`.
4. Offer up to three options and wait for explicit confirmation of one slot.
5. Use `Book Slot` to change that slot's status to `Booked`.
6. Use `Save Patient Appointment` to append the appointment record.
7. Confirm the booking only after both the slot update and appointment save succeed.

If the patient needs a person, the request is unclear, or a tool fails, the agent is instructed to use `Escalate to Human Staff`. These are the workflow's configured intentions; verify actual node behavior with test executions in your n8n environment.

## 7. Test before activation

Use a test Telegram bot and synthetic patient details wherever possible. Do not test with real patient data in a public or shared development environment.

- [ ] Send a normal message and verify the Telegram trigger receives it.
- [ ] Verify the non-urgent message reaches the Patient Intake Agent and a reply is sent.
- [ ] Test a supported language: English, Hindi, or Gujarati.
- [ ] Provide incomplete intake information and confirm the agent asks for the missing fields.
- [ ] Verify the agent offers only slots returned from the `Slots` sheet.
- [ ] Confirm a test slot and verify its status changes from `Available` to `Booked`.
- [ ] Verify exactly the expected test appointment data is appended to `Appointments`.
- [ ] Test an urgent-keyword message and verify the staff alert and patient reply are both delivered.
- [ ] Test a request for a human and verify escalation reaches the correct staff chat.
- [ ] Simulate a missing credential or unavailable sheet in a safe test environment and verify the workflow does not falsely claim that a booking succeeded.
- [ ] Inspect n8n execution logs and confirm they do not expose unnecessary personal information.
- [ ] Check the Google Sheets data and Telegram messages for unintended duplicates or incorrect slot updates.

Delete synthetic test records after testing, if appropriate for your test process.

## 8. Privacy, security, and operational notes

- Never commit API keys, bot tokens, OAuth tokens, private chat IDs, real patient details, or private Google Sheets links to this repository.
- Store credentials in n8n's credential manager, not directly in node fields or source files.
- Restrict workflow, execution-log, Telegram, and Google Sheets access.
- Review what patient information is sent to Telegram, Google, Gemini, and other connected services, and confirm the use complies with your organization's policies and applicable laws.
- Define a retention and deletion policy for appointment records and execution data.
- Have clinic staff review escalation wording, emergency procedures, and the department-routing rules.
- Do not use this example to diagnose, recommend medicines, provide treatment, or replace clinical judgment.
- Before production use, validate error handling and concurrent booking behavior. This workflow updates a slot row by `Slot_ID`, but the JSON alone does not establish that simultaneous booking attempts are protected against race conditions.

## 9. Troubleshooting

| Issue | Checks |
|---|---|
| Telegram trigger does not receive messages | Verify the Telegram credential, bot access, trigger configuration, and workflow activation state. |
| Staff alert is not delivered | Check the staff chat ID, bot membership and permissions, and Telegram node execution details. |
| Gemini node fails | Reconnect the Gemini credential, confirm model access, and inspect the node error in the execution. |
| No slots are returned | Check the document ID, exact `Slots` tab name, header spelling, department value, and `Available` status. |
| Slot is not updated | Confirm `Slot_ID` is unique and exactly matches the row, and the Google account can edit the sheet. |
| Appointment row is missing | Check the `Appointments` tab name and headers, Google Sheets permissions, and the execution result for both booking tools. |
| Agent claims success after a failure | Review the execution and agent/tool configuration. Do not use the workflow with real patients until this failure path has been corrected and tested. |

## 10. Before production

Do not activate this workflow for real patient use until credentials and placeholders are configured, all branches and tool failures have been tested, staff escalation is operational, and your organization has reviewed privacy, security, clinical safety, and applicable compliance requirements.


