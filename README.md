# IBD Companion

IBD Companion is a local-first OpenClaw skill for privately organizing personal
IBD/Crohn's disease records. It separates raw observations, treatment events,
tests, large review episodes, clinician-confirmed assessments, and personal
symptom baselines in a single SQLite database.

It is a recordkeeping and planning aid. It is not a diagnostic, prescribing,
or emergency-care tool.

## Highlights

- Append-only symptom observations with correction history
- Daily summaries calculated from raw logs rather than duplicated records
- Normalized food, sleep, stress, and other possible factors
- Injection scheduling, rescheduling, completion, and occurrence provenance
- Individual checkups linked to injections and/or large review episodes
- Atomic lab results plus versioned narrative reports
- Versioned, explicitly confirmed clinician assessments
- Draft, candidate, confirmed, and historical personal symptom baselines
- Calendar-month review plans with duplicate-safe occurrence generation
- Optional external reminder integration that stays off by default
- Additive SQLite migrations through schema V6

## Privacy model

The SQLite database is the sole source of truth and lives outside this
repository by default:

```text
${IBD_DB_PATH:-${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/private/ibd/ibd.sqlite3}
```

Do not commit databases, attachments, exported reports, real medical records,
or identifying health information. The included `.gitignore` blocks common
database and private-data paths, but it is not a substitute for reviewing every
commit.

## Requirements

- Python 3.10 or newer
- SQLite support from the Python standard library
- OpenClaw for skill-driven use; the command-line scripts also run directly

No third-party Python packages are required.

## Install

Clone the repository into the OpenClaw workspace skills directory:

```bash
git clone https://github.com/denniscriss/ibd-companion.git \
  ~/.openclaw/workspace/skills/ibd-companion
```

Initialize or migrate the private database:

```bash
python3 scripts/ibd_care.py init
```

To use a different database location, set `IBD_DB_PATH` for the command:

```bash
IBD_DB_PATH=/path/to/private/ibd.sqlite3 python3 scripts/ibd_care.py init
```

## Command overview

The base tracker handles symptoms, factors, injections, checkups, results, and
internal reminder rows:

```bash
python3 scripts/ibd_db.py --help
```

The care-context extension handles the disease profile, treatment and review
plans, review hierarchy, reports, confirmed assessments, and symptom baselines:

```bash
python3 scripts/ibd_care.py --help
```

Read `SKILL.md` before using the commands. It contains the safety boundaries,
source-of-truth rules, and explicit-confirmation requirements that keep unlike
medical meanings from being mixed together.

## Test

Tests use temporary databases and do not touch the default private database:

```bash
python3 -m unittest discover -s scripts -p 'test_ibd_*.py' -v
```

## Repository layout

```text
SKILL.md                 OpenClaw workflow and safety rules
references/schema.md     Data-model and migration documentation
scripts/ibd_db.py        Base SQLite tracker and CLI
scripts/ibd_care.py      Care-context extension and CLI
scripts/test_ibd_*.py    Automated tests using temporary databases
```

## Safety boundaries

- Preserve source wording and unknown values; do not invent medical facts.
- Never infer disease activity or causality from symptoms, metrics, or factor
  comparisons.
- Never change medication or treatment cadence from generated analysis.
- Record a clinician assessment only from an explicit confirmed source.
- Treat urgent or red-flag symptoms as a reason to seek professional care, not
  as a routine trend-analysis task.
- External notifications require separate user approval and configuration.

## License

MIT. See `LICENSE`.
