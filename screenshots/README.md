# Screenshot Capture Guide

Add these images manually after importing, configuring, and testing a synthetic-data copy. No image files are currently included, so the repository does not link to nonexistent screenshots.

| Expected filename | What to show |
| --- | --- |
| `workflow-overview.png` | The full main workflow with readable branch names |
| `successful-request.png` | A synthetic POST request and HTTP 200 response |
| `validation-error.png` | A missing-field or malformed-email request and HTTP 400 |
| `duplicate-response.png` | Repeated synthetic inquiry ID, HTTP 409, and no new inquiry row |
| `openai-classification.png` | The six structured output fields using synthetic input |
| `attention-notification.png` | An internal attention alert with synthetic customer details |
| `inquiries-sheet.png` | Demonstration inquiry rows, classifications, and review status |
| `workflow-log.png` | Completion records with notification outcomes |
| `error-handler.png` | The separately configured Error Trigger workflow and a redacted test outcome |

Before taking screenshots, hide:

- Credentials and credential identifiers.
- Webhook authentication secrets and private webhook URLs.
- Personal email addresses, account avatars, account names, and other account information.
- Google Sheet URLs and IDs, including the browser address bar and link previews.
- API keys, tokens, environment-variable values, and authentication headers.
- Real inquiry text, live execution data, workflow/instance identifiers, and private error details.

Use example.com addresses and synthetic request IDs in every visible test. Crop or permanently redact sensitive content, then reopen the saved image and inspect it at full size before committing. Do not rely on a reversible overlay or a hidden window to keep private information out of a capture.

Capture the corrected storage-failure response separately as evidence for R11 before describing this exact public artifact as fully tested. The main workflow's [release issue](../docs/test-results.md#release-verification-issue) and the optional nature of the error handler should remain clear in captions.

When a file exists, add a relative Markdown image link with a useful caption to the main README. Do not add placeholder image references before the corresponding PNG has been saved and reviewed.
