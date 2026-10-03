# Test Results

## Author-reported final regression suite

The project author reports completing the following 12-case regression suite using Postman and synthetic data, with every case passing in the tested environment. These results were supplied for the portfolio documentation; the packaging process did not rerun the live n8n, OpenAI, Google Sheets, or Gmail integrations. Selected reviewed screenshots are included. Execution exports and a reproducible Postman collection are not included.

| ID | Scenario | Expected outcome | Reported result |
| --- | --- | --- | --- |
| R1 | Normal sales inquiry | HTTP 200 and inquiry saved | PASS |
| R2 | High-priority billing | Human review and attention email | PASS |
| R3 | Human-review inquiry | Review routing | PASS |
| R4 | No-notification inquiry | `notification_status = not_required` | PASS |
| R5 | Duplicate request | HTTP 409 and no duplicate row | PASS |
| R6 | Missing required fields | HTTP 400 | PASS |
| R7 | Invalid email | HTTP 400 | PASS |
| R8 | Missing authentication | Rejected before normal processing | PASS |
| R9 | Spreadsheet formula-like input | Stored as literal text using RAW Sheets mode | PASS |
| R10 | Notification failure | Inquiry remains stored; `notification_status = failed` | PASS |
| R11 | Storage failure | HTTP 503; inquiry not marked successfully processed | PASS |
| R12 | Unexpected workflow failure | Execution fails; configured Error Handler is triggered | PASS |

All regression testing used synthetic data. No production or client data was used.

R12 requires a separately configured Error Trigger workflow. R9 concerns the `Save Inquiry` RAW append. R10 continues through the completion logger; failure of that later write is a separate condition.

## Included screenshot evidence

| Capture | What is visible | Evidence boundary |
| --- | --- | --- |
| [Successful request](../screenshots/successful-request-POSTMAN.png) | Synthetic inquiry, HTTP 200, and `not_required` | Supports the successful no-notification response |
| [Validation error](../screenshots/validation-error-POSTMAN.png) | Blank message rejected with HTTP 400 | Shows required-field rejection, not every validation rule |
| [Duplicate response](../screenshots/duplicate-response-POSTMAN.png) | HTTP 409 with a populated inquiry ID | Does not alone prove the spreadsheet row count was unchanged |
| [OpenAI output](../screenshots/openai-classification.png) | All six classification fields | One model result, not an accuracy benchmark |
| [Attention notification](../screenshots/attention-notification.png) | Human-review alert with summary and recommended action | Uses synthetic customer information |
| [Inquiries tab](../screenshots/inquiries-sheet.png) | Saved rows with high-priority and other-category review cases | Historical `REG-009` is outside this capture and needs its own retest to establish a fix |
| [Workflow Log](../screenshots/workflow-log.png) | Completion rows with `sent` and `not_required` | No `failed` notification outcome is shown |
| [Storage response](../screenshots/storage-failure-error.png) | HTTP 503, `not_saved`, and the storage-unavailable message | The distributed JSON still has the response-expression issue below |
| [Error Handler](../screenshots/error-handler-workflow.png) | Error Trigger, report preparation, and notification nodes | Shows the separate workflow structure, not proof of an error-triggered delivery |

The repository also includes two main-workflow overview images. No capture of Gmail failure with a retained inquiry row is included yet. These images are selected supporting evidence; they do not independently establish all 12 reported outcomes on the distributed artifact.

## Release verification issue

**R11 must be repeated on the imported public artifact after a response-expression correction.**

The packaged `503 - Storage Unavailable` node has a `responseBody` beginning with `{{ JSON.stringify(...) }}`. It lacks the leading `=` used for serialized n8n expressions, and that literal string is not valid JSON. Although the response-code setting is 503 and the error-output connections are intact, this is a substantive mismatch with the author-reported R11 outcome.

The static diagnosis is supported by n8n's [expression detection](https://github.com/n8n-io/n8n/blob/master/packages/workflow/src/expressions/expression-helpers.ts) and [JSON response handling](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/RespondToWebhook/RespondToWebhook.node.ts). The former requires an expression prefix; the latter parses literal response strings as JSON. This indicates the current body can fail before sending the intended storage-unavailable response. A live import and execution were not performed during packaging.

The packaging task required the supplied workflow to remain unchanged, so no functional correction was applied. The subsequently supplied storage-response screenshot shows a successful 503 response in the captured environment, but the distributed JSON still lacks the expression prefix. Correct and retest the imported public artifact before presenting that exact artifact as fully regression-verified.

## Packaging checks

The following checks are separate from the author-reported integration suite:

| Check | Result | Scope |
| --- | --- | --- |
| Public source inspection | PASS | 649 leaf values reviewed recursively; no prohibited private values found |
| JSON parsing | PASS | Public workflow, five synthetic requests, and three documentation JSON examples |
| Source preservation | PASS | Packaged workflow is byte-for-byte identical to the supplied public export |
| Graph integrity | PASS | 26 unique nodes, 27 connections, and 50 expression references to existing nodes |
| Configuration | PASS | Inactive workflow, Header Auth retained, three sheet placeholders and correct tab names, placeholder Gmail recipient |
| Notification and duplicate fixes | PASS | Exact `notification_status` field name on all three branches; 409 reads from `Normalize Inquiry` |
| Local logic checks | PASS | 22 synthetic checks of the exported validation code and mocked duplicate response |
| Documentation review | PASS | Public-file inventory and relative documentation/screenshot links checked; no nonexistent screenshot links |

Private-path ignore rules and staged-file privacy checks are also required before the initial commit. The local logic checks execute the exported JavaScript with mocked n8n inputs; they do not simulate the full n8n runtime.

These checks do not establish live service availability, model accuracy, Gmail delivery, n8n import compatibility on every version, or a successful end-to-end 503 response.

## Reproducing the suite

1. Follow [setup](setup.md), including the storage-response correction, and use a disposable test spreadsheet and internal test recipient.
2. Send the [sample requests](../data/sample_inquiries.json) one object at a time. Change IDs between new submissions; deliberately reuse one for R5.
3. For R6-R8, remove a required field, use a malformed email, or omit authentication respectively.
4. For R9, use harmless text such as `=1+1` as a subject and verify the stored cell remains literal text.
5. For R10-R11, deliberately break only the intended integration in a test copy, inspect HTTP and persistence outcomes, then restore access. Avoid affecting real accounts or live inquiry storage.
6. For R12, configure the separate handler and trigger an execution failure through the appropriate webhook execution mode.
7. Record the n8n version, test date, sanitized request/response, row outcome, and notification evidence for each case. No such environment details have been invented here.

Add redacted evidence using the [screenshot guide](../screenshots/README.md).
