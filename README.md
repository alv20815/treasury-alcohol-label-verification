# AI-Powered Alcohol Label Verification Prototype

**Deployed application:** [Open the prototype](https://g959119b7ca8101-emp.adb.us-ashburn-1.oraclecloudapps.com/ords/r/emp/ai-powered-alcohol-label-verification-app/cover-page)  
**Source repository:** [github.com/alv20815/treasury-alcohol-label-verification](https://github.com/alv20815/treasury-alcohol-label-verification)  
**Assignment instructions:** [treasurytakehome-rgb/instructions](https://github.com/treasurytakehome-rgb/instructions)  
**Design documentation:** [Approach, Tools, Assumptions, and Trade-offs](APPROACH_TOOLS_ASSUMPTIONS.md)

## Reviewer quick start

No local installation or Oracle APEX account is required to test the deployed prototype.

1. Open the deployed application, complete the brief cover prompt, and enter the prototype.
2. Open the **Reviewer Guide** to download the supplied test images, scenario catalog, batch manifests, and ZIP archives.
3. Start with **Demo label scenarios** for repeatable PASS, FAIL, and REVIEW outcomes, or run the **single-label**, **batch**, or **Live Test** workflow.
4. Open a result to review the field-by-field comparison, confidence, explanation, processing time, and export options.

The test assets are available inside the deployed prototype. For convenience, the same six test files are also packaged in these GitHub downloads:

- [`Test Files for Single Label and Live Test.zip`](Test%20Files%20for%20Single%20Label%20and%20Live%20Test.zip)
- [`Test Files for Batch Test.zip`](Test%20Files%20for%20Batch%20Test.zip)

## What the prototype does

The application treats **Application Data** as the authoritative baseline and compares it with information observed on alcohol-label artwork. It supports distilled spirits, wine, and beer and checks:

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

## Test workflows

### Deterministic scenarios

Twenty-three repeatable scenarios cover successful labels, field mismatches, missing data, warning defects, import-country cases, wine/beer ABV applicability, low-quality evidence, uncertain typography, and a non-alcohol control. These scenarios exercise the comparison engine without depending on an external service.

### Single-label verification

The reviewer enters Application Data and uploads one label image. The deterministic workflow and configured-provider workflow use the same stored comparison and decision logic.

### Batch verification

The batch workflow accepts a CSV manifest and ZIP archive. Supplied files support a 23-scenario acceptance batch and a 300-row test with expected totals of **60 PASS, 180 FAIL, and 60 REVIEW**.

### Live Test

Live Test sends an optimized or original copy of the uploaded image to the configured provider for pixel-based extraction. Expected Application Data is compared locally after extraction. Provider failures and unreliable evidence return **REVIEW** rather than being represented as a successful verification.

Use only synthetic or non-sensitive files in the public prototype.

## Results and exports

Each result shows expected and observed values, field status, confidence, diagnostic details, overall status, and server processing time. Random public tokens protect result links from simple sequential-ID access.

Individual results can be printed or exported as XLSX, JSON, or CSV. Batch evidence can be printed or exported as CSV or XLSX.

## Repository contents

```text
README.md
APPROACH_TOOLS_ASSUMPTIONS.md
f102.zip
f102_supporting_objects.sql
Test Files for Single Label and Live Test.zip
Test Files for Batch Test.zip
```

- `f102.zip` contains the current Oracle APEX Application 102 export, including the page definitions, shared components, Static Application Files, and reviewer-facing assets.
- `f102_supporting_objects.sql` creates the required `ALV_*` tables and view, compiles `ALV_VERIFICATION_PKG`, and seeds non-secret configuration and the 23 deterministic scenarios.
- The two test ZIP files provide convenient copies of the test assets that are also available from the deployed prototype.
- Provider secrets are intentionally excluded from the repository.

## Reproducing the application in Oracle APEX

The deployed application is the recommended evaluation path. The following steps are only for developers who want to reproduce the application in another Oracle APEX environment.

1. Create or select an Oracle APEX workspace with a clean parsing schema.
2. Open **SQL Workshop → SQL Scripts** and run `f102_supporting_objects.sql`.
3. Extract `f102.zip` to obtain the APEX application export.
4. Open **App Builder → Import**, import the extracted `f102.sql`, and complete the application installation.
5. For Live Test, create the required server-side APEX Web Credential and configure the provider settings in `ALV_CONFIG`. The API secret must remain in the Web Credential and must not be stored in SQL or source control.

Run the supporting-objects script in a clean schema because it creates the application database objects.

## Scope and limitations

- This is a standalone proof of concept and does not integrate with COLA.
- It assists human review; it is not a legal determination or a complete implementation of every beverage-label rule.
- Provider results can be affected by blur, glare, perspective, crop, small text, network conditions, or model behavior.
- The approximately five-second goal is a usability target, not a guaranteed service level.
- Synchronous live batches are capped at 25; production-scale processing of 200–300 images should use queued workers.
- Public access is intended only for synthetic or non-sensitive take-home data. Production would require authentication, authorization, audit, scanning, and monitoring.

## Contact

Questions or feedback: [Ruby.Tang@irs.gov](mailto:Ruby.Tang@irs.gov)
