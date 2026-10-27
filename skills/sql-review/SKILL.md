---
name: sql-review
description: Review, verify, and fix a SQL query, file, or the SQL in the current diff.
disable-model-invocation: true
argument-hint: "[query, file path, or object name — empty reviews the SQL in the current diff]"
---

Review the SQL in: $ARGUMENTS

1. **Find the query.** Inline SQL is the query. A path is read whole. An object name (view, model, procedure) is
   found in the repo first (models, migrations, `.sql` files), then in the catalog through access the repo
   already configures. Empty arguments → every `.sql` file and every SQL string changed in `git diff` and
   `git diff --staged`. Done when the full text of each query is in front of you, with its file and line.
2. **Load the method.** Invoke the `data-workflow:sql` skill and run its steps 1 to 5 (dialect, grain,
   correctness, performance, safety) on each query.
3. **Get a second read when it pays.** For a query over ~100 lines, a migration, or anything that writes,
   dispatch the `data-workflow:sql-reviewer` agent with the query, its dialect, and the grain you wrote down, and
   merge its findings into yours, keeping each once.
4. **Find out what can run.** Use only the access the repo or the shell already has: a configured CLI (`psql`,
   `dbt`, `databricks`, `bq`, `snow`), a notebook kernel, an MCP database tool. Credentials are never read or
   asked for in the chat. Without access, the review is static and the verification commands are written out for
   the user.
5. **Report.** Findings ranked by severity, in the `sql` skill's output format, then the fixed query in full.
6. **Verify.** Run the `sql` skill's verification on the original and the fix, read-only. A query that writes is
   verified on a rehearsal copy (transaction with rollback, clone, time travel) per the contract, never on the
   live target.
7. **Apply.** When the query came from a file, write the fix there and show the diff. Done when every finding is
   fixed and verified, or listed as open with the reason and the command that would close it.
