# Changelog

All notable changes to IBD Companion are documented here.

## Unreleased

- Prevent the base initializer from lowering a newer care-database schema
  version and replaying historical migrations.
- Add a regression test for preserving a V6 care database through base
  initialization, and document the schema-version policy and private backup
  workflow.

## v0.1.1 — 2026-08-06

- Replace drug-specific default metrics with a medicine-agnostic starter panel:
  CRP, ESR, HGB, WBC, PLT, and albumin.
- Add `metric-add`, `metric-list`, and `metric-deactivate` commands for safe,
  explicit metric-definition management without deleting historical results.
- Document the relational data-entry workflow and custom-metric rules.

## v0.1.0 — 2026-08-06

Initial public-ready release.

- Local SQLite tracking for IBD records, symptoms, possible factors, infusions,
  checkups, review episodes, reports, assessments, and symptom baselines.
- OpenClaw workflow guidance with explicit privacy and medical-safety boundaries.
- Bilingual English and Simplified Chinese documentation.
- Automated unit-test workflow for Python 3.10 and 3.12.
