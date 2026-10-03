# Security and Data Handling

This document describes controls visible in the public template and the responsibilities of anyone configuring it. It is not a claim of production certification or a guarantee that an LLM or external service cannot fail.

## Implemented controls

| Control | Implementation and boundary |
| --- | --- |
| Header-based authentication | `Webhook` keeps `headerAuth`; importers must supply their own Header Auth credential before execution |
| Required fields and types | `inquiry_id`, `submitted_at`, `name`, `email`, `subject`, and `message` must be present, nonblank strings |
| Email format | Basic non-whitespace address pattern; maximum 254 characters after trimming; not proof of address ownership or deliverability |
| Timestamp format | Timestamp pattern with seconds and a timezone, plus a `Date.parse` check; not a submission-age or strict calendar-validity policy |
| Inquiry ID | 3-64 characters using letters, digits, underscores, and hyphens after trimming |
| Maximum text lengths | Name: 100 characters; subject: 160; message: 4,000; checked before normalization |
| Normalization | Trim all six fields and lowercase the email address |
| Duplicate protection | Lookup by normalized inquiry ID before classification; non-atomic and therefore limited under concurrency |
| Structured model output | Strict six-field JSON Schema with controlled labels and no extra properties |
| Downstream AI validation | Required-field presence, empty-string checks, category/priority/sentiment allowlists, and Boolean review flag |
| Inquiry formula handling | `Save Inquiry` selects RAW spreadsheet input mode |
| Credential handling | Credentials are configured through n8n; the public JSON contains no credential objects or account references |
| Handled HTTP errors | Fixed error messages and validation field lists instead of exposing integration credentials or raw customer messages |

The downstream validator does not independently validate the type of every free-text classification field or enforce every relationship requested by the prompt. High priority and the review flag remain separate values. Unknown input fields are not explicitly rejected by the initial validator; normalization selects the six inquiry fields for subsequent processing.

## Authentication and credentials

Header Auth is intended for server-to-server use. Never put webhook secrets in public client-side JavaScript, screenshots, a public Postman environment, or a Git commit. Transport requests over HTTPS and grant integration accounts only the access they need.

Configure Header Auth, OpenAI, Google Sheets, and Gmail credentials inside n8n. The template contains only `YOUR_GOOGLE_SHEET_ID` and `notifications@example.com` as service placeholders. Its webhook path, model name, node UUIDs, assignment UUIDs, and condition UUIDs are structural configuration, not authentication secrets.

If a secret is accidentally committed, revoke or rotate it first. Removing a value from the newest file does not remove it from Git history. Review the exposed service and repository history before sharing again.

## Spreadsheet formula handling

`Save Inquiry` explicitly uses RAW input mode, serialized as `=RAW` in the export. Google documents that RAW stores values without parsing them; this is the control intended to keep formula-like inquiry text literal. See the [Sheets API input-mode reference](https://developers.google.com/workspace/sheets/api/reference/rest/v4/ValueInputOption).

`Prepare Safe Sheet Data` only copies fields. It does not escape formulas, and the append mappings read the combined inquiry values. RAW is not explicitly selected in `Log Processing`; this document does not claim equivalent protection for every log value or for data subsequently exported into another spreadsheet tool. Recheck interpretation when data crosses those boundaries.

## AI boundary and human oversight

The system prompt instructs the classifier to treat subject and message as data and ignore instructions embedded in them. Schema-constrained output and validation reduce the range of downstream values but do not guarantee semantic correctness or immunity to prompt injection.

The workflow does not grant the model tools or let it autonomously resolve a customer issue. Explicit routing decides whether an internal notification is required. A person evaluates the recommended action and handles the inquiry.

## Data leaving the workflow

- OpenAI receives the subject and message. A customer can include identifying information inside either field.
- `Inquiries` stores the customer name, email, original message, classification, and processing metadata.
- Gmail notifications include customer contact details, subject, classification, summary, and recommended action.
- `Workflow Log` can contain integration error text, including notification failure details.
- n8n execution history may retain request and response data according to the instance configuration.

Restrict access to these stores and recipients, and configure retention for the environment in which the workflow is used. The public sample requests are synthetic and use example.com email addresses only.

## Errors and operational scope

The custom 400, 409, and classification-validation 500 branches return limited error information. V1.2 corrects the 503 response expression, which returns the inquiry ID and a fixed storage-unavailable message instead of raw service errors. See [release verification](test-results.md#release-verification). Unexpected failures do not all use these custom responses; configure and review the separate error handler before operational use.

Google Sheets is appropriate here for demonstration and small-scale workflow storage. It is not a high-concurrency transactional database. Rate limiting, atomic duplicate protection, automatic retry recovery, and a comprehensive audit trail are outside this template's implemented controls.

## Public release checks

Packaging checks inspect the complete public file set and staged content for common secret patterns, private identifiers, real service links, and non-example email addresses. Documentation may discuss credentials and secrets without containing their values.

The repository includes only the sanitized workflow export. It omits credential objects, webhook/instance/version identifiers, the private workflow ID and error-workflow link, pinned customer executions, and deployment metadata. The workflow stays inactive. Node and internal field IDs remain to preserve the structure.

The storage-error Gmail screenshot has permanent pixel redactions over sender/account details, its execution ID, and its private execution URL. Public screenshots must not reveal live n8n account hostnames or workflow/execution identifiers even when the inquiry itself is synthetic. The original capture remains only in the ignored local `private/` directory.

`private/`, `secrets/`, `credentials/`, environment files, backups, and non-allowlisted workflow files are ignored by Git. No environment file is bundled. Ignoring a path is not encryption and does not protect an already-tracked file; review staged content before every public update.
