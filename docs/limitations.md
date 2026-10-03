# Limitations

## Release verification boundary

The v1.2 public export corrects the serialized expression prefix in `503 - Storage Unavailable`. The storage-response and received Gmail report captures support selected outcomes in the configured environment; the email still displays a v1.1 workflow label. Packaging checks did not run the complete live integration suite on the distributed v1.2 artifact. Verify the configured import before activation. Details are in [release verification](test-results.md#release-verification).

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
- No exact n8n version is pinned, and no reproducible Postman collection is bundled. Reviewed screenshots cover selected outcomes, including a received storage-error email; Gmail attention-notification failure evidence with a retained inquiry row is still absent. The 12 regression results are historical author-reported results, not a new v1.2 live integration run from packaging.

These boundaries describe the scope of a portfolio demonstration. The [future improvements](../README.md#future-improvements) outline possible extensions without claiming they already exist.
