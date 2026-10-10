# LeadPilot Lite - Manual Lead Email Writer

An n8n workflow that generates personalized cold email drafts and one follow-up from a supplied lead list, then saves them to Google Sheets for review. It does not send emails or source leads automatically.

## Workflow

POST Webhook -> Parse Lead List -> AI: Write Cold Email -> Merge Lead + Email -> Save to Google Sheets -> Respond: Success

## Features

- Validates sender details and up to 20 leads per request.
- Generates subject, email body and follow-up as JSON in one model call per lead.
- Uses only supplied lead and sender details in the prompt.
- Checks AI output shape and preserves lead-to-email pairing by position.
- Appends drafts to a review sheet with `Sent` set to false.

## Setup

1. Import `workflow.json` into n8n.
2. Connect an OpenAI credential or supported n8n AI Gateway connection to the AI node. The export uses `gpt-5.4-nano`; confirm availability in your account or choose a supported text model and check JSON output.
3. Connect Google Sheets credentials to the save node.
4. Replace `YOUR_GOOGLE_SHEET_ID` in the save node and the Respond: Success expression with your own spreadsheet ID.
5. Create a `Lite Pipeline` tab with these row-1 headers: `Name`, `Email`, `Title`, `Company`, `Subject Line`, `Email Body`, `Follow-up`, `Generated At`, `Sent`.
6. Execute the webhook test and POST `sample-request.json` to the test URL shown by n8n.
7. Check the saved row and response, then publish. Use your own production webhook URL for subsequent requests.

## Request example

```json
{
  "sender_name": "Test Sender",
  "sender_company": "Test Co",
  "value_prop": "We help small teams automate repetitive manual work.",
  "leads": [
    {
      "name": "TEST Fictional Lead",
      "email": "test.lead@example.com",
      "title": "Operations Manager",
      "company": "Example Corp (fictional test)"
    }
  ]
}
```

`sender_name`, `value_prop` and a non-empty `leads` array are required. Each lead needs `name` and `company`. Email, title, industry, company_description and notes are optional. sender_company and calendar_link are optional sender fields.

Example request after publishing:

```bash
curl -X POST 'https://YOUR_N8N_HOST/webhook/leadpilot-lite' -H 'Content-Type: application/json' --data-binary @sample-request.json
```

The configured success response includes `success`, `saved`, `leads` (lead, company, subjectLine), `sheetUrl` and `message`.

## Validation and limitations

A fresh editor test of the original configured workflow completed all six executable nodes. The generated subject, email body and follow-up were read back from Google Sheets, with Sent false. No email was sent. The actual HTTP status and response body were not independently verified by that editor test. This sanitized export needs configuration and testing in your own instance.

Review drafts before sending: model output can still contain inaccurate or unsuitable wording. There is no sending integration, lead sourcing, delivery tracking or unsubscribe handling in this workflow. The webhook currently has no authentication configured; before exposing your own endpoint, configure authentication and limit access to avoid unwanted requests and model charges. Avoid storing private lead data or credentials in this repository.

## Export contents

Credential references, private spreadsheet identifiers, pinned data and instance metadata have been removed. The sample request uses fictional details and example.com. The export is inactive until configured and published.
