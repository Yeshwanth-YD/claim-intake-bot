# Build spec: claim intake bot

Everything you need to decide is decided here, so each daily hour is pure building.

## Architecture

```
OneDrive /ClaimsInbox  --(Integration Service: file created)-->  DISPATCHER
   DISPATCHER: download PDF -> Document Understanding (digitise, extract)
              -> low confidence? -> Action Center validation task -> validated data
              -> Add Queue Item to "ClaimIntake" (Reference = ClaimNumber)
ORCHESTRATOR queue "ClaimIntake"  -->  PERFORMER (REFramework)
   PERFORMER: log in to legacy app -> enter claim -> read status message
              -> Successful / BusinessException / SystemException (retry)
   END: write run report CSV (claims processed, exceptions, seconds per claim)
```

## Document Understanding taxonomy

Document type: **Transport Accident Claim Form** (group: Claims, category: Intake)

| Field | Type | Required | Notes |
|---|---|---|---|
| ClaimNumber | Text | Yes | Format `CLM-2026-0000` |
| ClaimantName | Name/Text | Yes | |
| DateOfBirth | Date | Yes | Output as DD/MM/YYYY |
| AccidentDate | Date | Yes | Missing on claim 07 on purpose |
| AccidentLocation | Text | Yes | |
| VehicleRego | Text | Yes | |
| InjuryDescription | Text | Yes | |
| TreatmentProvider | Text | Yes | |
| InvoiceAmount | Number | Yes | Strip `$` and commas |

**Confidence rule:** if any required field is below **0.80** confidence, or is empty, send the document to validation (Action Center, or Validation Station when running attended). Claim 08, the noisy scan, is there to trigger this.

Extractor: use whatever your Studio/DU version offers best for a fixed-layout form (a Forms/template extractor, the generative extractor, or Regex Based Extractor as a fallback). The form layout never changes, so any of these works. Be ready to explain why you chose it.

## Queue item schema (`ClaimIntake`)

- **Reference:** the ClaimNumber (Unique Reference is on, so a duplicate document is caught at the queue, before the legacy app)
- **Priority:** Normal
- **Deadline:** today + 2 days (stands in for a claim SLA; mention this in interview)
- **SpecificContent:** the 9 taxonomy fields as strings, plus `SourceFile` and `ValidatedByHuman` (True/False)

Note: claim 06 reuses claim 02's number. Depending on run order, the queue rejects it at dispatch (unique reference) or the legacy app rejects it (ERR-201). Handle both, and be ready to say which layer you'd rather catch it in, and why.

## Performer (REFramework) settings

Data\Config.json, Settings (see docs\DAY1.md for the current keys):

| Name | Value |
|---|---|
| OrchestratorQueueName | ClaimIntake |
| LegacyApp_PathAsset | LegacyApp_Path |
| LegacyApp_CredentialAsset | LegacyApp_Credential |
| ReportFolder | C:\Users\rcdha\source\repos\claim-intake-bot\reports |

Constants: MaxRetryNumber = 0 in Config (the queue does retries, set to 2).

Workflows to add:

- `InitAllApplications`: open the HTML file in the browser, sign on with the credential asset.
- `CloseAllApplications`: Sign Off, close browser.
- `KillAllProcesses`: kill the browser process.
- `Process`: Clear, type the 9 fields, click Save, read the `#entryStatus` text.

## Exception matrix (read the status message after Save)

| Message from app | Meaning | Throw |
|---|---|---|
| `CLAIM ... SAVED` | success | none, set transaction Successful |
| `ERR-101` missing field | bad data | BusinessRuleException |
| `ERR-102`/`103`/`104` format | bad data | BusinessRuleException |
| `ERR-201` duplicate | already exists | BusinessRuleException |
| `SYSTEM ERROR 500` | app down | System Exception (queue retries; REFramework restarts apps) |
| no status within 15 s | app hung | System Exception |

Logging: one Info log per transaction with Reference and outcome; never log InjuryDescription, DateOfBirth or the full name (treat as personal and health information, even though this data is synthetic). This is a strong interview talking point.

## Selectors (legacy-style on purpose)

The app's fields are `f1` to `f9`, as cryptic legacy apps often are. Map them in the **Object Repository** with friendly names (ClaimNumber = f1 ... InvoiceAmount = f9). Buttons: `btnSignOn`, `btnSave`, `btnClear`, `btnList`, `btnSignOff`. Status text: `entryStatus`.

## Test plan (Day 5)

| Run | Setup | Expected |
|---|---|---|
| 1 | All 8 PDFs, normal mode | 6 saved (01–05, 08 after validation), 06 duplicate, 07 missing date |
| 2 | Tick "Simulate system outage" | Every item fails as system exception and retries; untick mid-run to see recovery |
| 3 | Tick "Slow mode" | All succeed; no timeouts |

Compare the app's Export CSV with `sample-claims\ground_truth.csv` to prove accuracy.

## Daily build order

- **Day 1:** Performer skeleton: InitAllApplications + CloseAllApplications working against the mock app; Object Repository mapped. Commit.
- **Day 2:** Process.xaml + exception matrix; test by manually adding 3 queue items in Orchestrator. Commit.
- **Day 3:** DU taxonomy + extraction on all 8 PDFs; compare with ground_truth.csv; confidence rule. Commit.
- **Day 4:** Dispatcher: Integration Service file trigger, DU, validation, Add Queue Item. Publish both, set the event trigger. Commit.
- **Day 5:** Test plan runs 1–3, report CSV, record the 2-minute video. Commit and tag v1.0.
