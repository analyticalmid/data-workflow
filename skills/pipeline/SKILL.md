---
name: pipeline
description: Design or review an ETL/ELT pipeline, from the sources to the tested output tables.
disable-model-invocation: true
argument-hint: "[what the pipeline must deliver, or a path to review]"
---

Pipeline: $ARGUMENTS

1. **Load the method.** Invoke the `data-workflow:etl-pipelines` and `data-workflow:data-quality` skills; on Databricks or
   Spark, also `data-workflow:databricks-spark`.
2. **Design or review.** A new pipeline runs the `etl-pipelines` steps 1 to 5. An existing one (a path, a DAG, a dbt
   project, a bundle) is read first: map its sources, tasks, and tables, then check each against the same steps.
3. **Write the design** in the chat, in this order:
   - Consumers and outputs: each output table with its grain, key, freshness, and layer.
   - Sources: each with its extraction method, change detection, delete handling, and limits.
   - Flow: bronze, silver, and gold tables, with the transformation between each pair.
   - Loads: each table's idempotent pattern and its backfill command.
   - Tests: each promise from the `data-quality` skill, where it runs, and what a failure does.
   - Schedule: trigger, interval, retries, timeouts, and alerts.
   - Personal data: where it enters, where it is minimized or masked, and how deletion requests reach every copy.
4. **Dispatch a review.** Send the design, or the code under review, to the `data-workflow:data-model-reviewer` agent and
   fold its findings in, keeping each once.
5. **Close the gaps.** Each open point becomes a question for the user or a decision stated with its trade-off. Done when
   every table has a grain, a load pattern, a backfill command, and its tests, and nothing is marked "to be defined".
6. **Build on request.** Code is written only after the user approves the design, following the repo's own conventions
   and the contract.
