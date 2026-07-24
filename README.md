# AI-Powered Alcohol Label Verification Prototype

**Deployed application:** [Open the prototype](https://g959119b7ca8101-emp.adb.us-ashburn-1.oraclecloudapps.com/ords/r/emp/ai-powered-alcohol-label-verification-app/cover-page)  
**Source repository:** [github.com/alv20815/treasury-alcohol-label-verification](https://github.com/alv20815/treasury-alcohol-label-verification)  
**Assignment instructions:** [treasurytakehome-rgb/instructions](https://github.com/treasurytakehome-rgb/instructions)  
**Design documentation:** [Approach, Tools, Assumptions, and Trade-offs](APPROACH_TOOLS_ASSUMPTIONS.md)  
**Last verified:** 2026-07-23

## Reviewer quick start

No local installation or Oracle APEX account is required to test the deployed prototype.

1. Open the deployed application, complete the brief cover prompt, and enter the prototype.
2. Open the **Reviewer Guide** for the recommended test sequence and downloadable test assets.
3. Start with **Demo Data / Sample Labels** for repeatable PASS, FAIL, and REVIEW scenarios that do not depend on an external provider.
4. Use **Alcohol Label Verification** or **Live AI Label Test** to compare one uploaded image with Application Data.
5. Use **Batch Upload / Batch Test** for the built-in 300-row test or the evaluator-supplied CSV-and-ZIP workflow.
6. Open a result to review expected and observed values, field status, confidence, explanations, timing, and export options.

## Test-file guide

The test assets are available from the deployed Reviewer Guide and from these repository downloads:

- [`Test Files for Single Label and Live Test.zip`](Test%20Files%20for%20Single%20Label%20and%20Live%20Test.zip)
- [`Test Files for Batch Test.zip`](Test%20Files%20for%20Batch%20Test.zip)

### Single-label and live tests

- `ALV_EXTENDED_TEST_IMAGES.zip` — optional sample-image pack. It is required only when using the supplied examples. **Unzip it locally**, then upload one individual PNG through the image-upload field.
- `demo_scenario_catalog.csv` — optional reference that maps scenario codes and Application Data baselines to image filenames and expected outcomes. It is not uploaded on the single-label or live-test page.
- `demo_contact_sheet.png` — optional visual guide to the supplied images. It is not uploaded on the single-label or live-test page.
- No supplied file is required when testing with your own JPEG, PNG, WEBP, or non-animated GIF.

The single-label and live-test pages accept **one image at a time**. They do not accept CSV, ZIP, PDF, Word, spreadsheet, or multi-file uploads.

### Uploaded batch tests

- 23-row test: upload `sample_batch_manifest_23.csv` with `sample_batch_images_extended.zip`.
- 300-row test: upload `sample_batch_manifest_300.csv` with `sample_batch_images_extended.zip`.
- Keep `sample_batch_images_extended.zip` **compressed** when uploading it.
- No files are required for the built-in 300-row test.

## What the prototype does

The application treats **Application Data** as the authoritative baseline and compares it with information observed on alcohol-label artwork. It supports distilled spirits, wine, and beer and checks:

- whether the uploaded document is an alcohol-beverage label;
- beverage category and alcohol-content applicability;
- brand name and class/type;
- alcohol by volume when required;
- net contents with unit normalization;
- bottler/producer name and address;
- country of origin for imports; and
- government-warning presence, exact wording, all-caps heading, bold heading, and legibility.

Field-specific rules allow reasonable case, spacing, punctuation, and unit differences where appropriate while keeping exact-compliance checks strict.

The overall decision is:

- **PASS** — all required checks matched;
- **FAIL** — a required check was missing or materially mismatched; or
- **REVIEW** — the evidence, confidence, typography, or provider response was not reliable enough for an automatic decision.

A confidently identified non-alcohol document returns **FAIL** for the alcohol-beverage document check. Alcohol-specific checks that cannot meaningfully be evaluated are shown as not evaluated rather than as invented field mismatches.

## Test workflows

### Demo Data / Sample Labels

Twenty-three repeatable scenarios cover:

- passing distilled-spirit, wine, beer, and imported-wine examples;
- explicit wine and beer ABV-not-applicable baselines;
- brand, class/type, beverage-category, ABV, net-content, producer/address, and import-country mismatches;
- missing ABV and missing government warning;
- warning wording, capitalization, boldness, and legibility problems;
- low-confidence, glare, unreadable-image, and uncertain-typography cases; and
- a non-alcohol control.

Demo scenarios use controlled observations to exercise the comparison and decision engine without depending on an external service. They do **not** claim to extract values from arbitrary uploaded pixels.

### Alcohol Label Verification

The reviewer enters Application Data and uploads one image. The page supports:

- deterministic sample scenarios; and
- live image analysis using the configured provider.

For an arbitrary image, document, painting, or evaluator-provided label, use **Live image analysis**. Deterministic scenarios should be paired with their corresponding supplied sample image.

Both execution paths persist results through the same PL/SQL comparison and decision engine.

### Batch Upload / Batch Test

The evaluator-facing batch workflow is deterministic and provides two complementary paths:

1. **Built-in 300-row test** — generates and processes the workload internally, with no file upload.
2. **Uploaded batch test** — reads an evaluator-supplied CSV manifest and compressed image ZIP, validates filename matching, persists each row, and processes each application independently using the row's deterministic profile.

The supplied 300-row built-in and uploaded tests intentionally use the same scenario distribution, so both should produce:

- **60 PASS**
- **180 FAIL**
- **60 REVIEW**

Matching totals are expected. The built-in test is the control; the uploaded test proves CSV parsing, ZIP handling, row mapping, persistence, and parity with that control.

The public batch page does **not** perform 300 live AI calls. Production-scale live image batches would require queued workers, controlled concurrency, retries, progress reporting, and cost controls.

### Live AI Label Test

Live AI Label Test sends an optimized or original copy of one uploaded image to the configured provider for pixel-based extraction. Expected Application Data is compared locally after extraction.

The page provides:

- distilled-spirits, wine, beer, and imported-wine presets;
- explicit ABV-applicability settings;
- Fast, Balanced, and Original-detail image-processing profiles; and
- direct navigation to the full Verification Result.

Provider failures and unreliable evidence return **REVIEW** rather than being represented as a successful verification.

Use only synthetic or non-sensitive files in the public prototype.

## Performance

The application reports two different timing measures:

- **Server Processing Time** — package validation, provider processing, comparison, and persistence.
- **Browser end-to-end time** — image preparation, upload, server processing, redirect, network transfer, and result-page rendering.

The stakeholder goal is approximately five seconds for normal single-label processing. Live timing varies with image complexity, provider latency, network conditions, and page rendering. Some live browser end-to-end runs can exceed the target; the application displays the measured result rather than hiding or redefining the target.

Deterministic batch timing is evidence of batch ingestion and comparison performance, not evidence of 300 live-provider image calls.

## Results and exports

Each verification result shows:

- expected and observed values;
- field status;
- confidence;
- an explanation for each comparison;
- overall PASS, FAIL, or REVIEW status;
- Server Processing Time;
- browser end-to-end time when available; and
- provider or diagnostic information when applicable.

Random public tokens protect result and batch links from simple sequential-ID access.

Individual results provide print and CSV, JSON, and XLSX export options.

Batch results provide:

- a compact printable summary suitable for **Save as PDF**;
- a batch-summary CSV; and
- complete row-level CSV and XLSX exports.

The compact batch print view summarizes the batch. Complete row-level evidence remains available in the CSV and XLSX exports.

## Repository contents

```text
README.md
APPROACH_TOOLS_ASSUMPTIONS.md
f102.zip
f102_supporting_objects.sql
Test Files for Single Label and Live Test.zip
Test Files for Batch Test.zip
```

- `f102.zip` contains the final Oracle APEX Application 102 export, including page definitions, shared components, Static Application Files, and reviewer-facing assets.
- `f102_supporting_objects.sql` creates the required `ALV_*` tables and view, compiles `ALV_VERIFICATION_PKG`, and seeds non-secret configuration and the 23 deterministic scenarios.
- The two test ZIP files provide convenient copies of the test assets that are also available from the deployed prototype.
- Provider secrets are intentionally excluded from the repository.

The application export and supporting-objects script must come from the same final release. A clean-schema installation should be completed before submission to confirm that the repository reproduces the deployed application.

## Reproducing the application in Oracle APEX

The deployed application is the recommended evaluation path. These steps are for developers who want to reproduce the application in another Oracle APEX environment.

1. Create or select an Oracle APEX workspace with a clean parsing schema.
2. Open **SQL Workshop → SQL Scripts** and run `f102_supporting_objects.sql`.
3. Extract `f102.zip` to obtain the APEX application export.
4. Open **App Builder → Import**, import the extracted `f102.sql`, and complete the application installation.
5. For live image analysis, create the required server-side APEX Web Credential and configure the non-secret provider settings in `ALV_CONFIG`.
6. Keep the provider API secret only in the APEX Web Credential. Do not place it in SQL, page items, JavaScript, documentation, or source control.

Run the supporting-objects script in a clean schema because it creates the application database objects.

## Scope and limitations

- This is a standalone proof of concept and does not integrate with COLA.
- It assists human review; it is not a legal determination or a complete implementation of every beverage-label rule.
- Provider results can be affected by blur, glare, perspective, crop, small text, network conditions, or model behavior.
- The approximately five-second goal is a usability target; live browser end-to-end performance is variable and is not guaranteed.
- The evaluator-facing 300-row batch workflow is deterministic rather than 300 live provider calls.
- Public access is intended only for synthetic or non-sensitive take-home data.
- Production deployment would require authentication, authorization, audit logging, malware scanning, rate limiting, monitoring, formal retention controls, and durable background processing for large live batches.

## Contact

Questions or feedback: [Ruby.Tang@irs.gov](mailto:Ruby.Tang@irs.gov)
