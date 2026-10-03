# Architecture

The public v1.2 template contains 26 nodes and 27 directed connections. One authenticated POST supplies one inquiry. OpenAI assists with classification; n8n applies the explicit validation, persistence, and routing rules.

## Main workflow

The diagram uses the node names from the JSON. Authentication is performed by `Webhook` before it emits an item.

```mermaid
flowchart TD
    webhook["Webhook: authenticated POST"] --> validate["Validate Inquiry"]
    validate --> valid{"Is Inquiry Valid?"}
    valid -->|"False"| badRequest["400 - Invalid Inquiry"]
    valid -->|"True"| normalize["Normalize Inquiry"]
    normalize --> lookup["Check Existing Inquiry"]
    lookup --> duplicate{"Is Duplicate?"}
    duplicate -->|"True"| conflict["409 - Duplicate Inquiry"]
    duplicate -->|"False"| classifier["OpenAI Classifier"]
    classifier --> combine["Combine Inquiry and Classification"]
    combine --> validateAi["Validate AI Output"]
    validateAi --> validAi{"Is Classification Valid?"}
    validAi -->|"False"| invalidAi["500"]
    validAi -->|"True"| prepareSheet["Prepare Safe Sheet Data"]
    prepareSheet --> save["Save Inquiry"]
    save -->|"Error output"| unavailable["503 - Storage Unavailable"]
    unavailable --> stop["Stop and Error"]
    save -->|"Success output"| attention{"Is Attention Needed?"}
    attention -->|"True"| prepareAlert["Prepare Attention Alert"]
    prepareAlert --> gmail["Send Attention Notification"]
    gmail -->|"Success output"| sent["Mark Notification Sent"]
    gmail -->|"Error output"| failed["Mark Notification Failed"]
    attention -->|"False"| notRequired["Mark No Notification Needed"]
    sent --> result["Prepare Processing Result"]
    failed --> result
    notRequired --> result
    result --> log["Log Processing"]
    log --> success["200 - Return Success"]
```

V1.2 corrects the expression prefix in `503 - Storage Unavailable`. A storage failure follows the error output to the HTTP 503 response, then `Stop and Error` marks the execution as failed. The connections are unchanged from v1.1. See [release verification](test-results.md#release-verification) for static checks and the boundaries of the captured runtime evidence.

## Data flow and decisions

| Stage | Behavior in the exported workflow |
| --- | --- |
| Intake | `POST customer-inquiry`, Header Auth enabled, response delegated to response nodes |
| Validation | Six required string fields; format and length checks; invalid input routes to 400 |
| Normalization | Trim all six fields; lowercase email |
| Duplicate lookup | Match the normalized ID against `Inquiry ID` in `Inquiries`; `alwaysOutputData` preserves the no-match path |
| Duplicate decision | Test for an existing `Inquiry ID` field in the lookup result; return the original normalized ID in the 409 response |
| Classification | Send subject and message to `gpt-4o-mini` using a strict six-field JSON Schema |
| Combination | Extract model output and combine it with data from `Normalize Inquiry`, independent of the lookup row shape |
| AI validation | Check required fields, allowed category/priority/sentiment, and Boolean review flag |
| Persistence | Append the inquiry and classification to `Inquiries`; RAW input mode is selected |
| Attention | Send an internal alert if priority is high OR the review flag is true |
| Completion | Carry `sent`, `failed`, or `not_required` to `Workflow Log`, then return 200 |

`Prepare Safe Sheet Data` copies values into `safe_*` fields; it does not escape formulas. `Save Inquiry` maps the original combined values, and RAW is the formula-interpretation control at that write boundary.

## Storage and logging

`Inquiries` stores the original message, classification, timestamps, processing status, human-review flag, and initial review status. The tab is a demonstration work queue, with no built-in reviewer assignment or resolution update.

`Workflow Log` receives completion records, including notification status and notification error text. Its `Error Message` column is present in the schema but has no value mapping in this export. Invalid requests, duplicates, invalid AI results, and storage failures do not pass through this completion logger.

The Gmail failure branch preserves the inquiry row and continues toward logging. A later log failure can still prevent the 200 response. Duplicate checking and append are separate operations and provide no atomic uniqueness guarantee.

## Optional separate error handler

This is a configuration pattern for a **separate workflow**, not another branch or bundled JSON file:

```mermaid
flowchart LR
    failure["Failed Automatic Execution"] --> trigger["Error Trigger"]
    trigger --> report["Prepare Error Report"]
    report --> administrator["Administrator Notification"]
```

Create or import the handler, configure its own credentials and recipient, then select it in this workflow's settings. The public template removes the original private handler reference. The handler should send a minimal internal report without exposing customer content or credentials.

Storage failures that reach `Stop and Error` can invoke this handler as well as unexpected execution failures. Error Trigger does not run for manual editor executions; test delivery through the production webhook of the activated/published main workflow. The [Gmail capture](../screenshots/storage-failure-error-notification.png) shows received storage-error reporting from a configured environment, not installation of the handler by the public template.

R12 in the [author-reported regression suite](test-results.md) depends on this separate configuration. No claim is made that a handler is installed merely by importing the public template.
