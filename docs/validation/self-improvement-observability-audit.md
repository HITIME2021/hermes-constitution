# Self-Improvement Observability Audit

Date: 2026-07-24

Status: `pass_with_followup`

Evidence source: operator-reported `Run-SELF-IMPROVE-001` summary. The full
local report was generated in WSL as `/tmp/self-improve-audit.md` during the
audit run.

## Summary

The audit confirmed that Hermes self-improvement is active and useful, but that
its patch evidence is not yet operator-visible enough for long-term trusted
governance.

Reported findings:

- 284 self-improvement patches.
- 16 patched skills.
- 0 operator-visible patch records.
- 3 high-risk skills accounted for 187 patches, about 66 percent of observed
  patch volume.
- 6 evidence gaps were reported. The most severe confirmed gaps were no
  git-backed version control for patched skill state and no durable diff audit
  trail.

## Interpretation

This is not a reason to disable self-improvement. Hermes self-improvement is a
learning mechanism and should remain available.

The governance gap is observability:

- The operator needs to know what changed.
- The patch must have before/after identity.
- The patch must have a visible diff.
- The patch must have a risk class.
- The patch must be reversible.
- High-risk patches must not become trusted authority without operator review.

## Required Policy Response

`docs/self-improvement-governance.md` now requires every applied
self-improvement patch to be attributable, diffable, classifiable,
operator-visible, and reversible.

Long self-improvement reports should be written to local evidence files. Chat
responses should return a compact summary and a local inspection path instead of
dumping the full report by default.

## Follow-Ups

- Implement durable patch records for Hermes self-improvement writes.
- Add local diff artifacts for patched skills.
- Add risk classification before a patch is treated as trusted.
- Quarantine or review already-applied high-risk patches that lack sufficient
  provenance.
- Keep full reports file-based and expose local `cat`, `less`, or `rg`
  commands for inspection.
