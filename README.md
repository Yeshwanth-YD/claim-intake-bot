# Claim Intake Bot (UiPath demo)

An end-to-end UiPath automation that takes insurance claim forms from a cloud folder, extracts the data with Document Understanding, sends uncertain fields to a person for validation, and enters each claim into a legacy claims system with no manual keying.

All data in this repo is synthetic. The insurer, people and claims are invented for testing.

## The problem

Claim forms arrive as PDFs, and staff key the details into an older claims system by hand. That is slow, error-prone and delays support for the person making the claim.

## How it works

<!-- Day 6: replace with a diagram image -->

1. A claim PDF is dropped into a OneDrive folder.
2. A UiPath **Integration Service** event trigger starts the dispatcher.
3. **Document Understanding** extracts 9 fields. Any field below 80% confidence goes to **Action Center** for a person to check.
4. Validated data becomes a **queue item** in Orchestrator, with the claim number as a unique reference and a deadline standing in for an SLA.
5. A **REFramework** performer logs in to the legacy app and enters the claim through GUI automation.
6. Business exceptions (missing field, duplicate claim) and system exceptions (system outage) are handled separately, with retries for system errors.
7. A run report shows claims processed, exceptions and time per claim.

## Results

<!-- Day 5: fill in from your test runs -->

| Measure | Result |
|---|---|
| Claims processed | |
| Field accuracy vs ground truth | |
| Average time per claim | |
| Exceptions correctly caught | |

## Design choices

<!-- Day 6: 3-5 bullets on why, e.g. queue unique reference vs app duplicate check, confidence threshold, no personal data in logs -->

## Repo layout

- `ClaimIntake.Dispatcher/` – trigger, Document Understanding, validation, queue
- `ClaimIntake.Performer/` – REFramework, legacy app entry
- `mock-legacy-app/claims.html` – the mock legacy claims system
- `sample-claims/` – 8 synthetic claim forms and `ground_truth.csv`
- `docs/SPEC.md` – build spec

## Demo video

<!-- Day 5: link to your 2-minute recording -->
