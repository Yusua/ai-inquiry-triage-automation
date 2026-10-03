# Test Results

## Historical author-reported regression suite

The project author previously reported completing the following 12-case regression suite using Postman and synthetic data, with every case passing in the tested environment. These results were supplied for the portfolio documentation; they are not a new full-suite result for v1.2. Packaging did not rerun the live n8n, OpenAI, Google Sheets, or Gmail integrations. Selected reviewed screenshots are included. Execution exports and a reproducible Postman collection are not included.

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
| [Storage response](../screenshots/storage-failure-error.png) | HTTP 503, `not_saved`, and the storage-unavailable message | Supports a captured storage-failure response; the v1.2 export is checked separately |
| [Storage-error email](../screenshots/storage-failure-error-notification.png) | Received Gmail report from `Stop and Error` for a synthetic inquiry | Supports delivery in the configured environment; the displayed workflow label is v1.1, so it is not proof of a complete v1.2 runtime test |
| [Error Handler](../screenshots/error-handler-workflow.png) | Error Trigger, report preparation, and notification nodes | Shows the separate workflow structure, not proof of an error-triggered delivery |

The repository also includes two main-workflow overview images. No capture of Gmail failure with a retained inquiry row is included yet. These images are selected supporting evidence; they do not independently establish all 12 reported outcomes on the distributed artifact.

## Release verification

**v1.2 corrects the previously identified 503 response-expression issue.**

The v1.1 public export's `503 - Storage Unavailable` response body began with `{{ JSON.stringify(...) }}` and lacked the leading `=` used for serialized n8n expressions. The v1.2 public export now begins with `={{`, retains HTTP 503, and preserves the storage-error route through `Stop and Error`. Its normalized node configuration and routing match the preceding public export apart from this response-expression correction.

The expression distinction follows n8n's [expression detection](https://github.com/n8n-io/n8n/blob/master/packages/workflow/src/expressions/expression-helpers.ts) and [JSON response handling](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/RespondToWebhook/RespondToWebhook.node.ts). Static export checks confirm the corrected prefix and preserved response setting. A local evaluation with a mocked `Normalize Inquiry` result also returns the intended JSON body, HTTP code 503, and storage-failure error message. This checks the serialized expressions, not the full n8n runtime; a live import and execution were not performed during packaging.

The supplied storage-response screenshot shows HTTP 503, and the newly supplied Gmail capture shows a received storage-error report. The email displays a v1.1 workflow label. These are supporting evidence from the captured environment, not a complete independent regression run on the distributed v1.2 artifact. Importers should still repeat R11 and the separately configured error-handler test against their own configured copy.

## Packaging checks

The following checks are separate from the author-reported integration suite:

| Check | Result | Scope |
| --- | --- | --- |
| Public source inspection | PASS | 649 leaf values reviewed recursively; no prohibited private values found |
| JSON parsing | PASS | Public workflow, five synthetic requests, and three documentation JSON examples |
| Release comparison | PASS | Normalized node configuration and routing match public v1.1 except for the corrected 503 response-expression prefix; private configuration is replaced with public placeholders |
| Graph integrity | PASS | 26 unique nodes, 27 connections, and 50 expression references to existing nodes |
| Configuration | PASS | Inactive workflow, Header Auth retained, three sheet placeholders and correct tab names, placeholder Gmail recipient |
| Notification and duplicate fixes | PASS | Exact `notification_status` field name on all three branches; 409 reads from `Normalize Inquiry` |
| Storage-response expressions | PASS | Correct `={{` prefix; mocked evaluation returns the original inquiry ID, `not_saved`, the fixed storage message, HTTP 503, and the intended `Stop and Error` message |
| Historical local logic checks | v1.1 evidence | 22 synthetic checks of the earlier export's validation code and mocked duplicate response; not rerun as a v1.2 suite |
| Documentation review | PASS | Public-file inventory and relative documentation/screenshot links checked; no nonexistent screenshot links |

Private-path ignore rules and staged-file privacy checks are also required before each public commit. The historical local logic checks execute exported JavaScript with mocked n8n inputs; they do not simulate the full n8n runtime.

These checks do not establish live service availability, model accuracy, Gmail delivery, n8n import compatibility on every version, or a successful end-to-end 503 response.

## Reproducing the suite

1. Follow [setup](setup.md) to configure the v1.2 public template, and use a disposable test spreadsheet and internal test recipient.
2. Send the [sample requests](../data/sample_inquiries.json) one object at a time. Change IDs between new submissions; deliberately reuse one for R5.
3. For R6-R8, remove a required field, use a malformed email, or omit authentication respectively.
4. For R9, use harmless text such as `=1+1` as a subject and verify the stored cell remains literal text.
5. For R10-R11, deliberately break only the intended integration in a test copy, inspect HTTP and persistence outcomes, then restore access. Avoid affecting real accounts or live inquiry storage.
6. For R12, configure and select the separate handler, activate/publish the main workflow in the test environment, and trigger an execution failure through its production webhook. Manual editor executions do not trigger Error Trigger; keep private webhook URLs and authentication headers out of captured evidence.
7. Record the n8n version, test date, sanitized request/response, row outcome, and notification evidence for each case. No such environment details have been invented here.

Add redacted evidence using the [screenshot guide](../screenshots/README.md).
