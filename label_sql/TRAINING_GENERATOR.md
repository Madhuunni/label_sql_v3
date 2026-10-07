# Schema-based training data generator

No pretrained models, PyTorch installation, database connection, or model API is needed to generate data. Run from the extracted project directory:

```bash
python -m label_sql.generate_training --schema data/my_schema.json --tables public.employees --per-column 1200 --out data/train.jsonl
```

Use `--tables public.employees public.departments` for several tables; omit `--tables` for all tables. IDs, unambiguous physical names, and table labels are accepted. Quote labels containing spaces. This limits target labels and join endpoints to selected tables; relationship paths can traverse intermediate tables from the full schema.

Each selected column label, including labeled JSON properties, receives exactly `--per-column` unique questions (minimum/default 1200). For example 9 labels produce 10,800 records. Labels in plans are table-qualified to avoid ambiguity. A question/plan pair is attributed to one target label; joins may mention other labels too.

Add `--include-joins` for cross-table projections between selected tables. Missing or ambiguous relationship paths are skipped for join examples; ordinary per-column generation continues. Self-join roles are not supported. Every emitted plan is checked by the existing compiler; this does not execute SQL or prove business correctness.

Use realistic values with `--sample-values data/training_values.example.json` (edit its keys to match your own schema). Values are synthetic defaults otherwise; text defaults are explicitly generic placeholders. Dates should be ISO strings and UUIDs valid UUID strings. Keys must match exact `Table label.Column label` references.

Output includes `question`, `plan`, `source`, and informational `metadata`. Existing training reads question/plan and ignores extra metadata. A sibling `train.jsonl.report.json` records count and operation-family coverage for each label. Parent directories are created; an existing output file is overwritten. The schema is never modified. The same schema/options/seed reproduce the output. Change `--seed` to change selection and ordering. Excessive counts beyond available unique combinations fail explicitly without writing a partial dataset.

## What 1200 means

These are distinct question strings built from template wording, supported operations, sample values, and explicit output-row limits. They are **not 1200 independently authored paraphrases or business intents**. Limits on aggregate queries limit result rows, not the rows being aggregated. Numeric/date filters, numeric aggregates, scalar sorting/distinct, null checks, counts and optional joins are covered. Unknown types and whole JSON values get conservative projection/null/count examples. JSON virtual columns use their declared scalar type. Complex grouping, HAVING, nested predicates, and business-specific rules still need curated examples.

Review generated data before training. Randomly splitting these closely related templates into train/validation/test would overstate generalization: use separately authored held-out questions and business scenarios. Increasing template count alone cannot establish model accuracy.

Example training (provide independently reviewed validation examples):

```bash
python -m label_sql.train --schema data/my_schema.json --train data/train.jsonl --validation data/validation.jsonl --output artifacts/label-sql
```

The original `bootstrap.py` remains available for small starter datasets.
