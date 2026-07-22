# Approach, Tools, Assumptions, and Trade-offs

## Approach

The prototype separates **observation** from **decision-making**.

1. Application-entered data establishes the expected baseline.
2. A deterministic scenario or configured vision provider supplies observed label values.
3. PL/SQL applies field-specific normalization and comparison rules.
4. Each check is stored with its expected value, observed value, status, confidence, and explanation.
5. The application calculates an overall PASS, FAIL, or REVIEW result and presents it as an explainable checklist.

This design keeps the final decision deterministic and auditable. The vision provider is used to extract observations from pixels; it is not asked to make the compliance decision.

## Architecture and tools

### Oracle APEX

Oracle APEX provides the public user interface, upload controls, validation, session state, navigation, reports, print/export actions, and deployment. The application is divided by workflow responsibility:

- cover and reviewer guidance;
- single-label verification;
- detailed result review;
- batch upload and batch results;
- deterministic scenarios; and
- live pixel analysis.

### Oracle SQL and PL/SQL

The `ALV_VERIFICATION_PKG` package centralizes:

- document and beverage-category checks;
- text, numeric, and unit normalization;
- warning-statement validation;
- PASS/FAIL/REVIEW precedence;
- provider orchestration and error handling;
- result persistence;
- batch processing; and
- deterministic test generation.

Keeping these rules outside the page definitions reduces duplicated logic and allows deterministic and live workflows to share the same comparison engine.

### APEX platform services

- `APEX_DATA_PARSER` reads uploaded CSV manifests.
- `APEX_ZIP` matches batch manifest rows to images in a ZIP archive.
- `APEX_WEB_SERVICE` sends live image requests to the configured provider.
- APEX Web Credentials keep provider secrets out of source code and browser content.
- APEX Data Export and browser print provide reviewer-friendly result output.

## Comparison strategy

Rules are intentionally field-specific:

- **Brand and class/type:** tolerate case, punctuation, and spacing differences.
- **ABV:** compare numerically using a configured tolerance and only when the application marks it applicable.
- **Net contents:** normalize compatible units before comparison.
- **Producer/address:** use stricter matching because omissions or material address differences can be significant.
- **Country of origin:** evaluate only when supplied as an application requirement.
- **Government warning:** evaluate presence, exact wording, heading capitalization, heading boldness, and legibility as separate checks.
- **Document/category classification:** fail confident non-alcohol or wrong-category evidence before treating individual fields as compliant.

Low-confidence classification, unreadable evidence, uncertain typography, and provider failures take precedence over ordinary mismatches and route the outcome to REVIEW where appropriate.

## Deterministic and live paths

The deterministic scenario catalog makes the core application testable without outbound access or provider credentials. It includes 23 scenarios across distilled spirits, wine, and beer and exercises positive, negative, missing-data, warning, applicability, uncertainty, and classification cases.

The Live AI Label Test sends actual uploaded image pixels to the configured provider. Fast and Balanced profiles resize images in the browser to reduce transfer and processing time; Original detail preserves the source image for difficult evidence. The returned structured observations are evaluated by the same PL/SQL rules used by the deterministic scenarios.

These two paths are complementary: deterministic tests provide reproducibility, while live tests demonstrate pixel-based extraction.

## Security and privacy design

- The public prototype is limited to synthetic or non-sensitive files.
- No provider secret is embedded in SQL, page source, JavaScript, screenshots, or repository files.
- Provider authentication is handled by a server-side APEX Web Credential.
- Verification and batch links use random public tokens in addition to record identifiers.
- Failed or unavailable provider calls produce REVIEW with diagnostics rather than silently falling back to a deterministic PASS.
- Global live-request limits reduce the risk of abuse of a public paid endpoint.
- APEX file-upload items use temporary-file storage. The public prototype should be used only with synthetic or non-sensitive files.

## Assumptions

- Application-entered values are the authoritative expected values.
- The uploaded artwork is intended to represent one alcohol-beverage label unless classification indicates otherwise.
- Distilled spirits require ABV. Wine or beer may mark ABV comparison not applicable only when the application record explicitly indicates that exception.
- Country of origin is checked when the application supplies an expected country.
- Case, punctuation, spacing, and compatible unit differences may be normalized for fields where those differences do not change meaning.
- Exact government-warning wording and presentation checks remain strict.
- Provider confidence, network availability, image quality, glare, cropping, and small text can affect live extraction quality and latency.
- The supplied test assets are synthetic and are not production regulatory records.

## Trade-offs and limitations

- The prototype is standalone and does not integrate with COLA or another production case-management system.
- Public no-authentication access improves evaluator convenience but is appropriate only for synthetic demonstration data.
- The live workflow is synchronous. It is suitable for individual tests and small batches, but production-scale 200-300 image processing should use queued workers, retries, rate limiting, and progress polling.
- Browser resizing improves latency but can reduce very small text detail; the Original detail option is available when higher fidelity is needed.
- The warning checks cover the assignment-focused wording and presentation signals, not every possible production typography, placement, contrast, or conspicuousness rule.
- Image enhancement such as perspective correction, glare removal, multi-image fusion, and advanced OCR preprocessing is outside the prototype scope.
- The application supports an explainable comparison workflow but does not replace an authorized human or legal determination.

## Performance approach

The application records both server processing time and reviewer-visible browser elapsed time. Deterministic scenarios isolate application and rule-engine performance. Live measurements include image preparation, network transfer, provider processing, and result persistence, so they vary by image profile and external conditions.

The design exposes those measurements directly and avoids presenting deterministic timing as proof of live-provider latency.
