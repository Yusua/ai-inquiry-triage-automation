# Screenshot Capture Guide

The following reviewed captures are included. They use synthetic inquiry data and illustrate selected outcomes from the configured environment. The workflow export remains generic and inactive.

| Included image | What it shows |
| --- | --- |
| [workflow-overview-1.png](workflow-overview-1.png) | Intake, validation, lookup, and classification |
| [workflow-overview-2.png](workflow-overview-2.png) | Storage, attention routing, notification outcomes, and completion |
| [successful-request-POSTMAN.png](successful-request-POSTMAN.png) | A synthetic request and HTTP 200 response |
| [validation-error-POSTMAN.png](validation-error-POSTMAN.png) | A blank-message request and HTTP 400 |
| [duplicate-response-POSTMAN.png](duplicate-response-POSTMAN.png) | HTTP 409 with the original inquiry ID |
| [openai-classification.png](openai-classification.png) | The six structured classification fields |
| [attention-notification.png](attention-notification.png) | An internal review alert with synthetic customer details |
| [inquiries-sheet.png](inquiries-sheet.png) | Saved demonstration rows, classification, and review status |
| [workflow-log.png](workflow-log.png) | Completion records with `sent` and `not_required` outcomes |
| [storage-failure-error.png](storage-failure-error.png) | HTTP 503 with `not_saved` in a captured run |
| [storage-failure-error-notification.png](storage-failure-error-notification.png) | A received Gmail storage-error report from `Stop and Error` for a synthetic inquiry |
| [error-handler-workflow.png](error-handler-workflow.png) | The separate Error Trigger workflow architecture |

Remaining evidence to add: a Gmail attention-notification failure capture showing `notification_status = failed` alongside the retained inquiry row. Use a filename such as `notification-failure.png` once that image exists.

Before taking screenshots, hide:

- Credentials and credential identifiers.
- Webhook authentication secrets and private webhook URLs.
- Personal email addresses, account avatars, account names, and other account information.
- Google Sheet URLs and IDs, including the browser address bar and link previews.
- API keys, tokens, environment-variable values, and authentication headers.
- Real inquiry text, live execution data, workflow/instance identifiers, and private error details.

Use example.com addresses and synthetic request IDs in every visible test. Crop or permanently redact sensitive content, then reopen the saved image and inspect it at full size before committing. Do not rely on a reversible overlay or a hidden window to keep private information out of a capture.

The included storage-response image shows HTTP 503 in its captured environment. The received Gmail report adds delivery evidence for the separate error handler, while its workflow label still reads v1.1. The v1.2 export's corrected response expression is checked separately; these captures do not establish a complete v1.2 regression run. The Error Handler image shows its structure. See [release verification](../docs/test-results.md#release-verification) for the distinction between static checks, received email evidence, and full runtime testing.

When a file exists, add a relative Markdown image link with a useful caption to the main README. Do not add placeholder image references before the corresponding PNG has been saved and reviewed.
