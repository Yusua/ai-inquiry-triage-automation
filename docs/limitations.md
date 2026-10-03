# Limitations

## Release issue requiring follow-up

The supplied `503 - Storage Unavailable` response body lacks n8n's serialized expression prefix. Correct and retest that path before relying on a controlled HTTP 503 response or presenting this exact export as fully regression-verified. The workflow was preserved without functional edits during packaging. Details and the author-reported R11 result are in [test results](test-results.md#release-verification-issue).

## Model judgment

- LLM classifications remain probabilistic; schema-valid output can still be misleading or incorrect.
- Human review is required for designated cases. Recommended actions are suggestions, not completed resolutions.
- Prompt instructions are not a complete defense against adversarial inquiry text.
- The downstream validator checks field presence, allowed labels, and a Boolean review flag; it does not duplicate the entire JSON Schema or enforce `high` priority implying `needs_human_review = true`.
- Attention routing uses high priority OR the review flag, while stored `Review Status` uses the flag alone. An inconsistent model result can therefore trigger an email while storing `Not Required`.

## Storage and reliability

- Google Sheets is not a transactional database. Duplicate lookup plus append is not atomic; simultaneous requests can race.
- OpenAI, Gmail, and Google Sheets can fail or become unavailable. A timeout can also make an external write's outcome uncertain.
- OpenAI API errors, lookup errors, and log-write errors have no dedicated recovery branch in this export.
- Gmail failure does not remove an already-saved inquiry. A later completion-log failure can still prevent the final 200 response.
- The completion log is not a record of every rejected request or workflow failure.
- RAW is selected for `Save Inquiry`, not explicitly for every Sheets node or later data-export destination.
- There is no automatic retry queue, dead-letter mechanism, or atomic retry/idempotency strategy.

## Operational scope

- No webhook-level rate limiting is included.
- No automatic reviewer assignment, SLA tracking, or escalation timer is included.
- No dashboard or multi-channel intake connectors are included.
- `Review Status` is an initial marker; there is no implemented resolution-update workflow.
- The error handler requires a separate workflow and configuration; it is not included in this repository.
- Credentials, the spreadsheet, sheet tabs, and notification recipient must be configured after import.
- No exact n8n version is pinned, and no reproducible Postman collection is bundled. Reviewed screenshots cover selected outcomes; Gmail-failure evidence and a completed Error Handler notification capture are still absent. The 12 regression results are author-reported, not new live integration results from packaging.

These boundaries describe the scope of a portfolio demonstration. The [future improvements](../README.md#future-improvements) outline possible extensions without claiming they already exist.
