# IBD Companion

[English](README.md) | [简体中文](README.zh-CN.md)

IBD Companion is an AI-assisted, local-first OpenClaw skill for privately
organizing personal inflammatory bowel disease (IBD) records, including
Crohn's disease and ulcerative colitis. It uses AI as a natural-language layer
over a structured local SQLite database, making it easier to record and query
the disease history and to prepare clearer information for clinical visits.

It is a recordkeeping and planning aid. It is not a diagnostic, prescribing,
or emergency-care tool.

## Core capabilities

1. **AI-assisted local IBD record.** Organizes the personal disease profile,
   biologic infusions and their linked lab results, annual or large reviews,
   and daily symptoms. Users can record or retrieve information through natural
   conversation instead of manually navigating database tables.
2. **Low-friction daily logging.** Symptoms and possible factors can be logged
   on demand, for example what was eaten and what symptoms followed. An
   optional daily reminder mode is planned; reminder delivery currently
   requires separate configuration and remains off by default.
3. **Personal pattern exploration (in progress).** Long-term symptom, food,
   sleep, stress, and other factor records can support exploratory comparisons
   of possible personal triggers. These results do not establish medical
   causality.
4. **AI-assisted checkup organization and visit preparation.** Keeps lab
   values, endoscopy, imaging, pathology, source reports, and confirmed
   clinician conclusions connected to the appropriate infusion or review.
   This makes the history easier to query and summarize before a clinical
   conversation.
5. **Treatment and review planning.** Tracks infusion schedules, changes,
   completion status, linked monitoring, and annual or other large-review
   plans without silently changing the treatment cadence.
6. **Local privacy and medical safety.** Real health data stays in a local
   SQLite database outside Git. The system supports recording, organization,
   and descriptive review, but does not diagnose, prescribe, or replace a
   clinician's judgment.

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

Repository releases (for example, `v0.1.0`) and the SQLite schema version
(currently V6) are separate: the release identifies a published code snapshot,
while the schema version controls safe migration of an existing private database.

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

The database is also created automatically on first use of either CLI if the
configured path does not exist.

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
