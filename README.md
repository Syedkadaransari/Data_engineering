# Data engineering learning log

Personal notes and small exercises for learning data engineering. Examples use invented data and generic architecture; they do not document an employer's systems.

## Start here

- [First onboarding entry: tracing a batch pipeline](learning-log/001-batch-pipeline-onboarding.md)
- [Entry template](templates/learning-entry.md)
- [Public-sharing guide](PUBLIC_SHARING.md)

## Initial structure

| File | Purpose |
| --- | --- |
| `README.md` | Scope, navigation, learning topics and next steps. |
| `PUBLIC_SHARING.md` | What is safe to publish and what must stay private. |
| `learning-log/001-batch-pipeline-onboarding.md` | First learning entry, generic pipeline walkthrough and synthetic practice task. |
| `templates/learning-entry.md` | Repeatable format separating understanding, practice, evidence and questions. |
| `.gitignore` | Exclude common credentials, local environments and generated files. |

## Topics

Layered data platforms; Airflow and Databricks orchestration; metadata-driven code generation; Git and CI/CD; data quality; slowly changing dimensions.

## Working approach

Write one short entry per learning goal. Explain the concept in my own words, practise with synthetic data, and record what was actually verified. Add runnable examples when they exist; a written exercise is not proof of a working pipeline.

## Next steps

1. Complete the synthetic customer exercise in entry 001.
2. Add a small local SQL or PySpark implementation with setup instructions.
3. Record duplicate-key, replay and historical-change test results.

Suggested repository description: **Public data engineering learning log: pipelines, Airflow, Databricks, Git/CI/CD, data quality and SCD2 using synthetic examples.**
