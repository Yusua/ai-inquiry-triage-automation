# Setup

Use a test n8n instance, a test spreadsheet, and synthetic data first. The public JSON imports inactive and contains placeholders. It does not configure accounts automatically.

## Prerequisites

- An n8n instance supporting the node versions in the export, including the OpenAI node with structured JSON Schema output.
- An OpenAI account with access to the configured `gpt-4o-mini` model.
- Google access that can edit your test spreadsheet and send Gmail notifications.
- Postman or another HTTP client that can send JSON and a private authentication header.

No pinned n8n release or reproducible Postman collection is included. Confirm node availability on your installation before running the workflow.

## 1. Import the template

Import [the public v1.2 workflow](../workflow/josh-inquiry-triage-v1.2-public.json) using n8n's workflow import-from-file option. Check that all nodes load without missing-node warnings. Keep it inactive while configuring integrations. Earlier public releases remain available in the repository's Git history.

The repository contains only this public export. Any configured export you later download belongs outside the public tree, for example in the ignored local `private/` directory. Git does not include that directory in a clone; create it locally if needed.

## 2. Configure authenticated intake

In `Webhook`, create or select a Header Auth credential through n8n. Choose a header name and a strong secret privately; configure the caller with the same values. Keep `authentication = headerAuth` enabled.

The node accepts POST requests at `customer-inquiry` and responds through the response nodes. Choose an unused path if another workflow on your instance already uses it. Use the URL displayed by n8n for the intended test or production mode; do not commit that private URL or the header value.

Header Auth is intended for server-to-server callers. Do not embed its shared secret in publicly delivered JavaScript. Use HTTPS for requests.

## 3. Configure OpenAI

Select your own OpenAI credential in `OpenAI Classifier`. Keep `gpt-4o-mini`, the existing prompt, and the strict six-field output schema. The public template includes no API key or account binding.

## 4. Configure Google Sheets

Create a new test spreadsheet and an n8n Google Sheets credential with edit access. Select that credential in all three nodes:

| Node | Document value to replace | Sheet tab |
| --- | --- | --- |
| `Check Existing Inquiry` | `YOUR_GOOGLE_SHEET_ID` | `Inquiries` |
| `Save Inquiry` | `YOUR_GOOGLE_SHEET_ID` | `Inquiries` |
| `Log Processing` | `YOUR_GOOGLE_SHEET_ID` | `Workflow Log` |

Use the same spreadsheet in every node. Document selection is By ID and tab selection is By Name. Replace the placeholder inside your configured n8n copy only; keep the distributed template generic.

Create the two tabs with the following headers in row 1. Spelling, spaces, and capitalization must match. The mappings use header names, so the schema's display order is not a separate requirement.

### Inquiries

Paste this tab-separated line into cell A1:

```text
Inquiry ID	Submitted At	Name	Email	Subject	Original Message	Summary	Category	Priority	Sentiment	Recommended Action	Human Review	Processing Status	Processed At	Review Status
```

### Workflow Log

Paste this tab-separated line into cell A1:

```text
Timestamp	Inquiry ID	Status	Step	Error Message	Notification Status	Notification Error
```

`Error Message` is reserved in the exported schema; the completion logger currently maps `Notification Error`, not a generic workflow-error record.

`Save Inquiry` selects RAW input mode, exported as `cellFormat: "=RAW"`. Confirm it resolves to RAW after import. RAW preserves submitted text instead of letting the Sheets API interpret it as entered formulas or typed values. This is distinct from the `Prepare Safe Sheet Data` node, which does not escape strings. See the [Sheets input-mode reference](https://developers.google.com/workspace/sheets/api/reference/rest/v4/ValueInputOption).

RAW is explicitly selected for the inquiry append. `Log Processing` has no explicit cell-format option in the template; do not assume this setting covers every spreadsheet write.

## 5. Configure Gmail

Select your Gmail credential in `Send Attention Notification`. Replace `notifications@example.com` with the intended internal recipient in your configured workflow. The alert includes customer contact fields and a summary, so use a recipient authorized to review that information.

## 6. Optionally configure an error handler

Create or import a separate workflow beginning with Error Trigger, prepare a minimal error report, and configure an administrator notification. Select that workflow as the Error Workflow in this workflow's settings. Configure its credentials separately.

The handler's JSON and private link are deliberately absent from this repository. Save the Error Workflow selection, then test with an automatic execution through the production webhook of your activated/published main workflow. Error Trigger does not run when you use Execute Workflow or the editor's test webhook. The error-handler workflow itself does not need activation/publication for Error Trigger. See n8n's [Error Trigger reference](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.errortrigger).

## 7. Verify the corrected storage response

V1.2 fixes the previous response-expression issue: `503 - Storage Unavailable` now has a response body beginning with the serialized expression prefix `={{`. Its HTTP code remains 503, and it is still connected to `Stop and Error`.

After import, confirm the body is in expression mode. Repeat R11 in a disposable configured copy and verify HTTP 503, `status = not_saved`, the original inquiry ID, and a failed execution at `Stop and Error`. When a separate error handler is linked, verify its Gmail delivery through the production webhook. See [release verification](test-results.md#release-verification); static checks and supplied screenshots do not replace a runtime test of your imported copy.

## 8. Send a synthetic test request

In Postman, choose POST, paste your private n8n test webhook URL, add the configured authentication header privately, set `Content-Type: application/json`, and use a raw JSON body:

```json
{
  "inquiry_id": "SAMPLE-001",
  "submitted_at": "2026-10-02T01:00:00Z",
  "name": "Alex Example",
  "email": "alex@example.com",
  "subject": "Quotation for a small-team subscription",
  "message": "Could you provide pricing and onboarding options for a team of five? We are comparing services for next month."
}
```

Additional cases are in [sample_inquiries.json](../data/sample_inquiries.json). The file is an array for convenience; send one object at a time. Use a new ID for each new inquiry, then resend an existing ID to exercise the duplicate branch. Send both requests sequentially for this test.

Inspect the HTTP response, `Inquiries` row, initial review status, notification outcome, and `Workflow Log` row. Model wording and classifications may vary; validate the output contract and routing rather than expecting identical text.

## 9. Verify failures and activate

Repeat the [12 regression scenarios](test-results.md) against your configured copy. Use a disposable test setup for Gmail and storage failure simulation, then restore its working configuration. Verify formula-like text remains literal in `Inquiries`, authentication rejects an unauthenticated caller, and the separate error handler runs when configured.

Activate/publish your configured copy for controlled production-webhook error-handler testing, then finish the regression checks before operational use. Keep credentials, live execution data, and screenshots containing private information out of Git.

## Publishing the repository manually

When GitHub CLI is unavailable, the local repository can be published after review:

1. Sign in to GitHub and create a **public** repository named `ai-inquiry-triage-automation` under your account.
2. Use this description: `AI-assisted customer inquiry triage workflow using n8n, OpenAI, Google Sheets, Gmail, webhook validation, human-review routing, and error handling.`
3. Leave the remote empty: do not initialize another README, license, or .gitignore.
4. Open a terminal in this repository and replace `YOUR-USERNAME` in the commands below with your GitHub account name. Authenticate through your normal Git credential flow; do not put a token in the URL.

```bash
git remote add origin https://github.com/YOUR-USERNAME/ai-inquiry-triage-automation.git
git push -u origin main
```

Before pushing an update, review [release verification](test-results.md#release-verification), stage only the sanitized public export, and inspect every new screenshot for private data. Add these topics through the repository's About settings: `n8n`, `openai`, `automation`, `workflow-automation`, `ai-automation`, `google-sheets`, `gmail`, `webhooks`, `api`, `customer-service`.
