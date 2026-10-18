# 001 — Tracing a batch pipeline from source to consumption

Date: 2026-10-18  
Status: Concept notes and planned synthetic exercise; no pipeline executed.

## Goal

Explain how a batch moves through data layers, how orchestration differs from processing, and how to validate a change before making it available to consumers.

This is a generic teaching model. Layer names and responsibilities vary between platforms.

## 1. Follow the data through the layers

| Layer | Purpose | Synthetic customer example |
| --- | --- | --- |
| Source | Supplies the input | An invented daily customer snapshot. |
| Raw | Preserves received input and ingestion metadata | Keep original values plus batch date and arrival time. |
| Curated | Applies agreed types, validation and standardisation | Parse customer IDs and dates; isolate invalid records. |
| Business model | Represents business entities and history | Maintain customer address history using SCD2 where needed. |
| Consumption | Serves a defined user need | Produce counts of current customers by city. |

DQ checks belong at relevant boundaries. A successful raw load alone does not establish that the consumption output is correct.

## 2. Separate orchestration from processing

Airflow can coordinate dependencies, readiness checks, retries and task status. A task can submit a Databricks job, where Spark performs the transformations.

In this exercise, assume the submitting task waits for the Databricks run to finish and propagates its failure. Successful submission alone must not count as successful processing.

Example sequence: check input readiness, load raw data, validate and curate, update history, then refresh the consumption result. Each downstream step depends on successful completion of its prerequisites.

If the source is late, inspect readiness and the affected batch before retrying. If processing fails, inspect both the Airflow task and the Databricks run. Replaying a batch should not create duplicate history.

Some ingestion designs use data, metadata and a readiness marker. In this example, a marker signals that delivery is complete; the loader still checks the expected files, schema and batch date. This is an illustrative convention, not a universal requirement.

## 3. Understand code generation and delivery

A metadata-driven generator such as cgen can reduce repetitive ETL or DML authoring. Generated output still needs review: the generator cannot decide whether the intended business behaviour is correct.

For a public exercise, write original SQL or PySpark rather than copying internal generator output or commands.

Generic delivery workflow:
1. Update the local default branch and create a feature branch.
2. Make a focused change and inspect any generated diff.
3. Run applicable formatting, static checks and meaningful validation.
4. Review the staged diff for unintended or sensitive content.
5. Commit locally, push the branch and open a pull request.
6. Address review feedback and merge after required checks pass.
7. If deployment is configured, verify the deployed job and data results.

A commit is local; pushing sends it to the remote. Merging code, deploying a job definition and loading data are separate events. CI/CD behaviour depends on the repository's configured workflows.

## 4. Practise DQ and SCD2 with invented data

Use these two complete snapshots:

| Snapshot date | customer_id | city |
| --- | --- | --- |
| 2026-01-01 | C001 | Delft |
| 2026-01-01 | C002 | Leiden |
| 2026-01-02 | C001 | Gouda |
| 2026-01-02 | C002 | Leiden |

Assumptions:
- customer_id is the business key; city is the only tracked attribute.
- Changes take effect at midnight on the snapshot date.
- Validity intervals are half-open: valid_from is included and valid_to is excluded.
- NULL valid_to means the current version.
- Missing customers do not imply deletion in this exercise; deletion handling needs a separate rule.
- Out-of-order batches need a separate correction strategy and are outside this first exercise.

Expected history after both batches:

| customer_id | city | valid_from | valid_to | is_current |
| --- | --- | --- | --- | --- |
| C001 | Delft | 2026-01-01 | 2026-01-02 | false |
| C001 | Gouda | 2026-01-02 | NULL | true |
| C002 | Leiden | 2026-01-01 | NULL | true |

The changed city expires C001's old version and creates a new one. C002 remains unchanged. Expiring and inserting should be handled atomically so a failure cannot leave history incomplete.

### Practice steps

1. Create the two snapshots locally from the table above.
2. Preserve the raw rows and batch metadata.
3. Reject or quarantine null keys and duplicate keys within a snapshot before updating history.
4. Load the first snapshot, then apply the second.
5. Compare the output with the expected history table.
6. Replay the second batch; the history must still contain exactly three rows.
7. Try a duplicate C001 row and a null customer_id. Confirm that the invalid batch cannot silently change the business output.

### Validation checklist

| Check | Expected result |
| --- | --- |
| Required columns and types | Match the exercise's input contract. |
| Keys within each snapshot | Non-null and unique. |
| Current versions | Exactly one per customer in this exercise. |
| History intervals | No overlaps; closed intervals have valid_from < valid_to. |
| Unchanged customer | C002 has one history row. |
| Replay | No additional history rows or changed validity dates. |
| Consumption after second batch | Gouda: 1 current customer; Leiden: 1. |
| Failed DQ gate | Downstream publication does not proceed. |

## Verification record

Not run yet. The tables above are expected results, not observed test output. This entry contains no runnable DAG, job, generator configuration or implementation.

## Questions to explore

- How should late or corrected snapshots change existing history?
- Which DQ failures should block publication, and which may be quarantined?
- How can a partially failed batch be recovered safely?

## Next step

Implement the two-snapshot example locally in SQL or PySpark, then add reproducible commands and actual validation results.
