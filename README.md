# AI-Powered Alcohol Label Verification Prototype

**Deployed application:** [Open the prototype](https://g959119b7ca8101-emp.adb.us-ashburn-1.oraclecloudapps.com/ords/r/emp/ai-powered-alcohol-label-verification-app/home)  
**Source repository:** [github.com/alv20815/treasury-alcohol-label-verification](https://github.com/alv20815/treasury-alcohol-label-verification)  
**Assignment instructions:** [treasurytakehome-rgb/instructions](https://github.com/treasurytakehome-rgb/instructions)  
**Design documentation:** [Approach, Tools, Assumptions, and Trade-offs](APPROACH_TOOLS_ASSUMPTIONS.md)

## Reviewer quick start

No local installation or Oracle APEX account is required to test the deployed prototype.

1. Open the deployed application.
2. Start with **Demo label scenarios** to see repeatable PASS, FAIL, and REVIEW outcomes.
3. Run the **single-label**, **batch**, or **Live Test** workflow.
4. Open a result to review the field-by-field comparison, confidence, explanation, timing, and export options.

Page 6 is the reviewer guide and provides the supplied test images, scenario catalog, batch manifests, and ZIP archive.

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

- **PASS** - all required checks matched;
- **FAIL** - a required check was missing or materially mismatched; or
- **REVIEW** - the evidence, confidence, typography, or provider response was not reliable enough for an automatic decision.

## Test workflows

### Deterministic scenarios

Twenty-three repeatable scenarios cover successful labels, field mismatches, missing data, warning defects, import-country cases, wine/beer ABV applicability, low-quality evidence, uncertain typography, and a non-alcohol control. These scenarios test the comparison engine without depending on an external service.

### Single-label verification

The form accepts Application Data and an uploaded image. Reviewers can select a deterministic Demo mode or, when configured, the external provider. Both paths use the same stored comparison and decision logic.

### Batch verification

The batch workflow accepts a CSV manifest and ZIP archive. Supplied files support a 23-scenario batch and a 300-row acceptance test with expected totals of **60 PASS, 180 FAIL, and 60 REVIEW**.

### Live Test

Live Test sends the uploaded image copy and extraction instructions to the configured provider. Expected Application Data is compared locally after extraction. Fast and Balanced profiles resize the upload copy to a maximum long edge of 768 or 1024 pixels; Original detail sends the full image.

Provider failures and unreliable evidence are returned as REVIEW rather than being represented as a successful verification. Use only synthetic or non-sensitive images in the public prototype.

## Results and exports

Each result shows expected and observed values, field status, confidence, diagnostic details, overall status, and server processing time. Random public tokens protect result links from simple sequential-ID access.

Individual results can be printed or exported as XLSX, JSON, or CSV. Batch evidence can be printed or exported as CSV or XLSX.

## Repository contents

The repository is intentionally small:

```text
README.md
APPROACH_TOOLS_ASSUMPTIONS.md
f102.sql
```

`f102.sql` is the current Oracle APEX Application 102 export generated from Oracle APEX 24.2.17. It contains the page definitions, shared components, Static Application Files, and reviewer-facing assets. The Static Application Files are embedded in the export and do not need to be uploaded separately.

## Running or importing the application

The recommended evaluation path is the deployed URL above; reviewers do not need to install Oracle APEX or import the application.

For source review, open `f102.sql` directly in GitHub. An Oracle APEX developer can import it through **App Builder -> Import**. The export expects the corresponding `ALV_*` database objects and `ALV_VERIFICATION_PKG` to exist in the target parsing schema. Provider credentials are intentionally excluded from the repository and must remain in a server-side APEX Web Credential.

## Live-provider configuration

Provider settings are read from `ALV_CONFIG`. The API secret remains in a server-side APEX Web Credential and is intentionally excluded from SQL, page source, screenshots, and this repository. The Live Test page displays its current readiness before a request is submitted.

## Scope and limitations

- This is a standalone proof of concept and does not integrate with COLA.
- It assists human review; it is not a legal determination or a complete implementation of every beverage-label rule.
- Provider results can be affected by blur, glare, perspective, crop, small text, network conditions, or model behavior.
- The approximately five-second goal is a usability target, not a guaranteed service level.
- Synchronous live batches are capped at 25; production-scale 200-300 image processing should use queued workers.
- Public access is intended only for synthetic or non-sensitive take-home data. Production would require authentication, authorization, audit, scanning, and monitoring.

## Contact

Questions or feedback: [Ruby.Tang@irs.gov](mailto:Ruby.Tang@irs.gov)
