# Case Study

## Problem

Customer inquiry handling involves more than reading a message. Someone needs to check that the submission is usable, understand its purpose, decide whether it is urgent, record it, and make sure the right person sees it. When those steps are manual, duplicate messages and inconsistent review decisions become hard to track.

For my first automation portfolio project as a Computer Engineering student, I built a focused webhook workflow that makes those decisions and outcomes visible. The objective was a working inquiry-triage demonstration with clear failure paths, not an autonomous customer-support agent.

## Initial Approach

I started with deterministic mock classification rules to establish the request structure, expected responses, and storage flow. This made it easier to reason about the workflow before introducing a model whose wording and judgments can vary. The final exported workflow uses OpenAI; the mock classifier is not included in this release.

Google Sheets provided an inspectable destination for the demonstration. It let me review the original inquiry beside the structured classification and notification result without adding a separate application or database interface.

## Development

The workflow developed around three boundaries: accepting a valid request, accepting a usable classification, and recording an outcome even when an optional notification fails. I kept the normalized inquiry available by explicit node references so that a lookup result or model response could not become the only source of customer data.

OpenAI now receives the inquiry subject and message and returns six schema-defined fields. n8n validates that result, saves the record, and applies a separate attention rule. The model's recommendation is advisory; storage and routing follow deterministic conditions.

## Technical Challenges

| Challenge | Why it mattered |
| --- | --- |
| Google Sheets returning no items | A lookup with no match could stop the new-inquiry path before classification |
| Preserving original data | A Sheets row and an OpenAI response have different shapes from the submitted inquiry |
| Duplicate detection | Repeated IDs needed to exit before creating another inquiry record |
| Replacing mock rules with OpenAI | Model output needed an explicit contract rather than free-form text |
| JSON Schema and validation | A structured response still needed downstream checks before storage |
| Priority versus human review | A low-priority case can require judgment, while negative sentiment alone does not establish urgency |
| Notification status handling | Sent, failed, and unnecessary email paths needed consistent output fields |
| HTTP response codes | Callers needed distinct outcomes for invalid input, duplicates, and processing failures |
| Gmail and storage failures | An email failure after persistence differs from failing to save the inquiry |
| Error Trigger configuration | Unexpected failures need a separate workflow and an explicit settings link |
| Spreadsheet formula handling | User-supplied text can resemble a formula |
| Authentication and regression tests | A useful demo must cover unauthorized requests and failure paths, not only successful submissions |

## Solutions

`Check Existing Inquiry` enables `alwaysOutputData` so a no-match lookup can continue. `Is Duplicate?` inspects the Sheets `Inquiry ID` field, while the duplicate response reads the original ID from `Normalize Inquiry`. This avoids assuming the lookup row still contains `inquiry_id`.

`Combine Inquiry and Classification` explicitly retrieves the original normalized data, extracts the model result, and returns both together. The OpenAI request uses strict JSON Schema output, and `Validate AI Output` checks presence, controlled labels, and the review Boolean. That separates the AI boundary from the storage decision.

The attention condition is high priority OR a true human-review flag. All three notification outcomes set the same `notification_status` field and converge on `Prepare Processing Result`. Gmail's error output records a failed notification without deleting the inquiry that was already saved.

`Save Inquiry` has a separate error-output path toward a storage-unavailable response and `Stop and Error`. An optional n8n Error Trigger workflow can notify an administrator about unexpected execution failures. The public package removes the private link and documents how to configure a replacement.

RAW writes at `Save Inquiry` address formula interpretation without rewriting the original inquiry text. Header authentication and validation guard intake, while credentials remain configured through n8n instead of being distributed in the public export.

The project regression report covers 12 synthetic scenarios, including duplicate submissions, authentication failure, formula-like input, email failure, storage failure, and error-handler triggering. [Test results](test-results.md) preserves that author-reported record and clearly separates it from new packaging checks.

## Final Result

The result is a 26-node AI-assisted inquiry workflow with structured classification, review routing, demonstration storage, internal notifications, and explicit response branches. The public repository provides the sanitized template, five synthetic requests, import instructions, architecture, and documented boundaries without publishing the configured instance or account details.

Packaging review of v1.1 found a response-expression mismatch in its 503 node. V1.2 corrects the serialized expression prefix while preserving the routing and intended response fields. The updated portfolio also includes a redacted Gmail storage-error report from the configured environment. [Release verification](test-results.md#release-verification) distinguishes this static correction and captured notification delivery from a complete runtime regression of the public v1.2 template.

## Lessons Learned

I learned to inspect the data emitted by each integration instead of assuming it preserved the original payload. Explicit references, clear field names, and small validation steps made the routing easier to reason about.

I also learned that model output, notification delivery, and successful persistence are different outcomes. Keeping those states separate produces more useful HTTP responses and logs. The same principle applies to public release work: valid JSON, absence of private data, and successful runtime behavior are separate checks.

The next engineering step would be stronger persistence and idempotency, followed by better operational evidence and observability. Those are future improvements; this project remains a deliberately scoped automation demonstration.
