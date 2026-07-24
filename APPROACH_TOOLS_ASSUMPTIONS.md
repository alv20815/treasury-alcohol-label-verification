# Approach, Tools, Assumptions, and Trade-offs

## Current implementation

The prototype provides two complementary verification paths:

1. **Deterministic scenarios and deterministic batch tests** provide repeatable PASS, FAIL, and REVIEW evidence without relying on an external service.
2. **Live image analysis** sends the pixels from one uploaded image to a configured vision provider, receives structured observations, and compares those observations with the application baseline.

Both paths use the same PL/SQL comparison and decision engine. Deterministic scenarios are test fixtures; they do not claim to read arbitrary uploaded pixels. Live analysis is the path used for an evaluator-supplied image, document, or artwork.

## Approach

The design separates **observation** from **decision-making**.

1. Application-entered data establishes the expected baseline.
2. A deterministic scenario or the live vision provider supplies observed values.
3. `ALV_VERIFICATION_PKG` applies field-specific normalization, applicability, comparison, confidence, and decision rules.
4. Each check is persisted with its expected value, observed value, result, confidence, and explanation.
5. The application calculates an overall `PASS`, `FAIL`, or `REVIEW` decision and displays an explainable row-by-row checklist.

The live provider is used only to extract and classify visible evidence. It receives image pixels and extraction instructions; the expected application values are compared afterward in PL/SQL. The provider is not asked to make the final compliance decision.

## Application structure

Oracle APEX separates the prototype by workflow responsibility:

- **Reviewer Guide** — entry point, project documentation, test files, and recommended test order.
- **Alcohol Label Verification** — guided single-label workflow with application data and one uploaded image.
- **Verification Result** — authoritative field-by-field outcome, timing, diagnostics, image link, print, and export actions.
- **Batch Test** — built-in and uploaded deterministic batch testing with summary and row-level results.
- **Approach, Tools, and Assumptions** — in-application design and limitation documentation.
- **Demo Data / Sample Labels** — catalog of 23 deterministic scenarios.
- **Live AI Label Test** — dedicated real-pixel extraction workflow with application presets and image-processing profiles.

## Architecture and tools

### Oracle APEX

Oracle APEX provides the public user interface, uploads, validation, session state, navigation, reports, Static Application Files, print/export actions, and deployment.

### Oracle SQL and PL/SQL

`ALV_VERIFICATION_PKG` centralizes:

- alcohol-document classification and beverage-category comparison;
- brand and class/type normalization;
- ABV applicability and numeric comparison;
- net-content parsing and unit conversion;
- producer/address and import-country comparison;
- government-warning presence, wording, capitalization, boldness, and legibility checks;
- `PASS` / `FAIL` / `REVIEW` precedence;
- provider orchestration and error handling;
- verification, result, and batch persistence;
- deterministic scenario generation; and
- built-in and uploaded batch processing.

Keeping the rules in one package reduces page-level duplication and ensures that deterministic, uploaded-batch, and live workflows use the same decision logic.

### APEX platform services

- `APEX_DATA_PARSER` reads uploaded CSV manifests.
- `APEX_ZIP` locates images referenced by batch-manifest rows.
- `APEX_WEB_SERVICE` sends live image requests to the configured provider.
- APEX Web Credentials store provider authentication outside page source, PL/SQL source, and the repository.
- `APEX_DATA_EXPORT` and browser print support reviewer evidence downloads.
- APEX temporary-file storage receives uploads before the application validates and processes them.

### Current live-provider configuration

The deployed release uses an OpenAI vision-capable model through a server-side APEX Web Credential. The provider, model, endpoint, image detail, timeout, and confidence thresholds are configuration values rather than secrets embedded in the source. The current deployed model is shown in the Live AI Label Test status panel.

## Beverage categories and applicability

The prototype supports these application categories:

- `DISTILLED_SPIRITS`
- `WINE`
- `BEER`

Distilled spirits always require an ABV comparison. Wine and beer may mark ABV as not applicable only when the application record explicitly says that the field is not required. The prototype does not independently determine whether a legal exemption applies.

Country of origin is evaluated when the application supplies an expected country. Domestic examples without an expected country return `NOT_APPLICABLE` for that check.

## Comparison and decision strategy

Rules are field-specific rather than using one generic string comparison:

- **Alcohol-document check:** distinguishes an alcohol-beverage label from an unrelated document or image.
- **Beverage category:** compares distilled spirits, wine, and beer separately from the broad document classification.
- **Brand and class/type:** tolerate permitted case, punctuation, and spacing differences.
- **ABV:** compare numerically using the configured tolerance and only when applicable.
- **Net contents:** normalize compatible units before comparison.
- **Producer/address:** use stricter matching because a missing company, city, state, or other material component may be significant.
- **Country of origin:** compare when supplied by the application baseline.
- **Government warning:** evaluate presence, exact wording, heading capitalization, heading boldness, and legibility as separate checks.

Decision precedence is conservative:

- a confident non-alcohol classification produces `FAIL` for the document check and marks alcohol-specific checks as not evaluated;
- provider failures, unreliable extraction, low confidence, or uncertain typography produce `REVIEW` where an automatic conclusion is unsafe;
- a required missing value or material mismatch produces `FAIL`; and
- `PASS` requires all applicable required checks to match.

## Deterministic scenarios

The deterministic catalog contains 23 scenarios across distilled spirits, wine, and beer. It covers:

- positive examples;
- brand, class/type, category, ABV, net-content, producer/address, and country mismatches;
- missing ABV and missing warning cases;
- warning wording, capitalization, boldness, and legibility problems;
- wine and beer ABV-not-applicable examples;
- low confidence, glare, unreadable evidence, and uncertain typography; and
- a non-alcohol control.

These scenarios are controlled test fixtures. They make the comparison engine reproducible in restricted-network environments and should not be confused with live pixel extraction.

## Batch-test approach

The evaluator-facing Batch Test page is intentionally deterministic and offers two paths:

1. **Built-in 300-row control** — creates the 300-row workload internally and requires no uploaded files.
2. **Uploaded CSV/ZIP parity test** — reads an evaluator-supplied manifest and compressed image ZIP, validates the files, maps manifest rows to ZIP entries, persists the applications, and runs each row using its supplied deterministic profile.

The supplied 300-row manifest intentionally uses the same scenario distribution as the built-in control. Therefore, both paths are expected to produce the same totals:

- 60 `PASS`
- 180 `FAIL`
- 60 `REVIEW`

Matching totals are evidence that the uploaded CSV/ZIP ingestion path reaches the same comparison engine correctly. The two paths differ in ingestion work, mode, and elapsed time—not in the intended business outcomes.

Page 3 does **not** represent 300 live AI calls. Production-scale live processing of 200–300 images would require durable background jobs, controlled concurrency, retries, rate limiting, progress reporting, and cost controls.

## Live image-analysis approach

Live AI Label Test accepts one JPEG, PNG, WEBP, or non-animated GIF. The evaluator may use one of the supplied synthetic images or upload another non-sensitive image.

The page provides three processing profiles:

- **Fast** — resizes the image for the lowest practical latency.
- **Balanced** — preserves more detail for small label text.
- **Original detail** — sends the source image when difficult evidence requires the highest available fidelity and accepts the additional latency.

The provider returns a structured classification and extracted label fields. The application then applies the same PL/SQL checks used by deterministic scenarios. A painting, flyer, age-verification screen, or other non-label document should fail the broad alcohol-document check; alcohol-specific fields are shown as not evaluated rather than being presented as arbitrary mismatches.

## Test-file handling

The supplied files have different purposes:

- `ALV_EXTENDED_TEST_IMAGES.zip` is an optional sample-image pack for single-label and live testing. It must be unzipped locally, and one individual image is uploaded.
- `demo_scenario_catalog.csv` is an optional reference that maps profiles, baselines, expected outcomes, and image filenames.
- `demo_contact_sheet.png` is an optional visual guide.
- `sample_batch_manifest_23.csv` and `sample_batch_manifest_300.csv` are manifests for uploaded deterministic batch tests.
- `sample_batch_images_extended.zip` must remain compressed when uploaded with either batch manifest.

No files are required for the built-in 300-row test.

## Performance approach

The application records and displays two separate measures:

- **Server Processing Time** — package-level validation, provider processing when applicable, comparison, persistence, and result creation.
- **Browser end-to-end time** — reviewer-visible elapsed time including client image preparation, upload, request processing, redirect, network transfer, and result-page rendering.

Deterministic timings demonstrate application and rule-engine responsiveness but are not presented as proof of live-provider latency. Live performance varies with image dimensions, processing profile, network conditions, provider load, and the visual complexity of the upload.

The assignment's approximately five-second target remains a usability target. The current live workflow is operational, but repeated median and p90 browser measurements should be reported separately from server timing. The prototype does not claim that every live request will complete within five seconds.

## Security and privacy design

- The public deployment is intended only for synthetic or non-sensitive files.
- No API secret is stored in SQL source, page source, JavaScript, screenshots, or repository files.
- Provider authentication is handled by a server-side APEX Web Credential.
- Public verification and batch links use random tokens in addition to numeric identifiers.
- Provider failures return `REVIEW` with diagnostics and never silently fall back to a deterministic `PASS`.
- Uploads enter through APEX temporary-file storage and may be copied into verification or batch records for reviewer evidence. Retention is configuration-driven; production would require an approved records-retention and deletion policy.
- Public no-authentication access is a take-home convenience and is not a production authorization model.

## Assumptions

- Application-entered values are the authoritative expected values.
- A live upload normally represents one label or one document to classify.
- Distilled spirits require ABV; wine or beer may mark ABV not applicable only when the application baseline explicitly says so.
- Country of origin is required only when supplied as an application expectation.
- Case, punctuation, spacing, and compatible unit differences may be normalized only where they do not change meaning.
- Government-warning wording and presentation checks remain strict.
- Confidence, image quality, crop, glare, perspective, small text, provider behavior, and network conditions can affect live results and latency.
- The supplied assets are synthetic demonstrations, not production regulatory records.

## Trade-offs and limitations

- The prototype is standalone and does not integrate with COLA or another production case-management system.
- It supports human review but does not replace an authorized legal or regulatory determination.
- The live evaluator workflow is synchronous and single-image. It is not a production queue.
- Browser resizing improves latency but may reduce very small text detail; the Original detail profile is available when fidelity is more important than speed.
- Warning checks cover the assignment-focused wording and presentation signals, not every possible typography, placement, contrast, size, or conspicuousness rule.
- Perspective correction, glare removal, multi-image fusion, advanced OCR preprocessing, malware scanning, production audit integration, and enterprise monitoring are outside the take-home scope.
- Public access, tokenized links, and synthetic data are appropriate for a prototype but not sufficient for production privacy, authorization, records management, or incident response.

## Packaging and deployment boundary

The reproducible release separates database objects from the APEX application:

1. `f102_supporting_objects.sql` creates the required `ALV_*` schema objects, compiles `ALV_VERIFICATION_PKG`, and seeds non-secret configuration and the deterministic scenarios.
2. `f102.zip` contains the APEX Application 102 export and reviewer-facing application assets.
3. The provider Web Credential is created manually in the target APEX workspace so the secret never enters source control.

A clean installation should compile the package specification and body as `VALID`, report no `USER_ERRORS`, and contain 23 active deterministic scenarios before live-provider configuration is enabled.
