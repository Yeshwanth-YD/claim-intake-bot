# Claim Intake Bot · UiPath end-to-end demo

A UiPath automation that reads insurance claim forms (PDF) with **Document Understanding**, checks them against business rules, queues valid claims in **Orchestrator**, and enters them into a **legacy claims system** with a **REFramework** robot. No manual keying.

> All data in this repo is synthetic. The insurer, people and claims are invented for testing.

**Demo video (2 min 12 sec):

https://github.com/user-attachments/assets/6d83595c-7c33-4fe2-8ad7-0b38f161f1c3





---

## The problem

Claim forms arrive as PDFs, and staff key the details into an older claims system by hand. That is slow, error-prone, and delays support for the person who made the claim.

## How it works

```mermaid
flowchart LR
    A[Claim PDFs] --> B[Dispatcher]
    B --> C[Document Understanding<br/>generative extraction, 9 fields]
    C --> D{Business-rule<br/>validation}
    D -- fails --> E[needs-review folder<br/>for a person]
    D -- passes --> F[(Orchestrator queue<br/>ClaimIntake)]
    F -- duplicate reference --> G[Rejected at queue]
    F --> H[Performer<br/>REFramework]
    H --> I[Legacy claims app<br/>GUI automation]
    I -- SAVED --> J[Successful]
    I -- ERR-xxx --> K[Business exception<br/>not retried]
    I -- app down --> L[System exception<br/>screenshot, restart, retry]
```

**Dispatcher** (`ClaimIntake.Dispatcher`)
1. Reads each claim PDF with **Extract Document Data** (generative extractor, 9 fields described in plain English).
2. Validates every field: required fields present, claim number pattern, DD/MM/YYYY dates, accident date not in the future, date of birth before the accident, numeric invoice amount.
3. Valid claims go to the `ClaimIntake` queue, with the claim number as a **unique reference**. Invalid claims are copied to `needs-review/` with the reason logged.

**Performer** (`ClaimIntake.Performer`, 2026 REFramework)
1. Signs in to the legacy app using an **Orchestrator credential asset**, never a password in code or config.
2. Enters each queued claim, then reads the app's status message to decide the outcome.
3. **Business exceptions** (bad data, duplicates) are marked and not retried. **System exceptions** (app down, element missing) trigger a screenshot, a clean restart of the app, and a queue retry.

## Results

<!-- RESULTS -->
From the recorded end-to-end run (8 claim PDFs):

| Measure | Result |
|---|---|
| Claims entered into the legacy app with no manual keying | 6 of 6 valid claims |
| Duplicate claim | rejected at the queue, never reached the app |
| Claim with a missing accident date | routed to a person, with the reason logged |
| Simulated outage mid-run | system exception, screenshot, app restarted, claim retried and saved |
| Dispatcher time (extract + validate + queue) | 8 PDFs in 2 min 35 sec, about 19 sec each |
| Performer time (including the outage and restart) | 6 claims in 2 min |
| Extraction on clean forms | all fields checked matched the ground truth |
| Extraction on the skewed, noisy scan | 2 character-level errors from speckle noise (`Sofià` for `Sofia`, `Bayside Physio : Clinic`) |

The noisy-scan errors are well-formed values, so format rules can't catch them. That is the case for routing scanned or low-quality documents to a person, sample-based QA, or a trained extractor with real confidence scores.

## Design decisions

- **Two layers of duplicate protection.** The queue's unique reference stops duplicates before any robot time is spent; the legacy app's own check is the backstop.
- **Business vs system exceptions.** Bad data goes to a person and is never retried; technical failures are retried after a clean restart. `MaxRetryNumber` stays 0 because retries are owned by the queue.
- **No confidence score from generative extraction.** Generative extractors don't return calibrated confidence, so validation uses business rules instead. Asking the model twice was considered and rejected: it doubles cost and latency for little gain on a fixed-layout form. For a stable, high-volume form in production, a trained extractor with real confidence scores plus sample-based QA would be the next step.
- **Found by deliberately breaking things.** Interfering with the app mid-run exposed a browser restart race condition (fixed with a recovery delay) and a typing failure being misread as bad data (next fix: *Verify execution* on every Type Into, so a failed keystroke becomes a retried system exception rather than a business exception).
- **Claim number on every log line.** Each transaction's logs carry its claim number, so support can trace any failure straight to the claim.
- **No personal data in logs.** Logs carry the claim number only; names, dates of birth and injuries never appear.
- **Legacy selectors in one place.** The mock app's cryptic field IDs (`f1`–`f9`) are mapped once, so a UI change is a single fix.

## Run it yourself

1. Create a free UiPath Community account and enable **Document Understanding** for your tenant (Admin > Tenants > Services).
2. In Orchestrator's **Shared** folder, create the queue `ClaimIntake` (unique references on, 2 retries) and the assets `LegacyApp_Credential` (`demo` / `demo123`) and `LegacyApp_Path` (full path to `mock-legacy-app/claims.html`).
3. Run `ClaimIntake.Dispatcher` (PDFs into the queue), then `ClaimIntake.Performer` (queue into the legacy app).
4. To test recovery, tick **Simulate system outage** in the mock app's yellow test panel mid-run.

## Repo layout

| Path | What it is |
|---|---|
| `ClaimIntake.Dispatcher/` | PDF extraction, validation, queueing |
| `ClaimIntake.Performer/` | REFramework robot for the legacy app |
| `mock-legacy-app/claims.html` | Mock legacy claims system with business rules and test switches |
| `sample-claims/` | 8 synthetic claim forms, including a messy scan, a duplicate and a missing field, plus `ground_truth.csv` |
| `needs-review/` | Where claims that fail validation are routed |

## Next (version 2)

- **Integration Service trigger:** start the dispatcher automatically when a claim PDF lands in OneDrive.
- **Action Center:** replace the `needs-review` folder with validation tasks for a person.
