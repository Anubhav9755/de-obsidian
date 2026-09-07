# DE Notes Vault

My learning notes for the 6-Month Data Engineer Roadmap + AWS CCP.

## Structure
- `domains/` — AWS CCP domain notes (conceptual)
- `sql/` — SQL concepts, one file per topic, code blocks inline
- `python/` — Python concepts (decorators, concurrency, etc.)
- `pyspark/` — PySpark concepts and code
- `airflow/` — Airflow DAGs, concepts, gotchas
- `dbt/` — dbt models, tests, concepts
- `projects/` — weekend project logs, one file per project, decisions + what broke
- `daily/` — optional quick scratch notes, doesn't need to be permanent

## Convention
Every topic note follows this shape:
1. **What it is** (1-2 lines, plain language)
2. **Why it matters** (the actual use case)
3. **Code / example**
4. **Gotchas / things I got wrong**

Link related notes with `[[note-name]]` — e.g., link `sql/joins-and-subqueries.md`
from `domains/domain1-cloud-concepts.md` when a concept ties back to SQL practice.

## Sync
This vault auto-commits/pushes via the Obsidian Git plugin every ~10-15 min.
Manual push: `Ctrl/Cmd+P` → "Obsidian Git: Commit and push"
