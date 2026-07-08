---
feature_area: grantmaking-pss
status: duplicate-of-catalog
catalog_id: P8
catalog_ref: patterns/canonical/org-metadata-extraction-audit.md
note: "This is the literal source file for P8 (Org metadata extraction & audit workflow). Do not re-catalog."
---

# Funding Award - Object

## Look-Ups

| Field Name | API Name | Data Type | Required/Active |
| :--- | :--- | :--- | :--- |
| Owner Name | `OwnerId` | Lookup(User,Group) | True |
| Last Modified By | `LastModifiedById` | Lookup(User) | False |
| Created By | `CreatedById` | Lookup(User) | True |
| Signed By | `GPSG_Signed_By__c` | Lookup(User) | True |
| Individual Application | `IndividualApplicationId` | Lookup(Proposal) | False |
| Program | `ProgramId` | Lookup(Program) | False |
| Funding Opportunity | `FundingOpportunityId` | Lookup(Funding Opportunity) | True |
| Contract | `ContractId` | Lookup(Contract) | False |
| Contact | `ContactId` | Lookup(Contact) | False |
| Grantee Signed By | `GPSG_Grantee_Signed_By__c` | Lookup(Contact) | True |
| Budget | `BudgetId` | Lookup(Budget) | False |
| Application Form | `ApplicationFormId` | Lookup(Application Form) | False |
| Awardee | `AwardeeId` | Lookup(Account) | |

## Status Lifecycle

### 1. Draft
- **Action**: Generate Award
  - Template: `docGenerationSample` / `fndSingleDocxLwc` / English
- **Action**: Send Via Email Action

### 2. Negotiation
- **Action**: Submit for approval

### 3. In Approval Process
- **Process Name**: Agreement Approval
- **Unique Name**: `Agreement_Approval`

### 4. Grantee Signing
- *Note: No signature feature built yet*

### 5. Agency Signing
- *Note: No signature feature built yet*
- **Action**: Flow: `GPSG Funding Award - Congressional Notifications`

### 6. Awarded
- **Action**: Flow: `Funding Award After Update`
  - *Description*: Looks at record status for Awarded and changed record type to awarded if it is not already.

## Extraction Summary

**What was found:**
- **Objects**: Extracted the standard `FundingAward` object including all `GPSG_` custom fields, plus the PSS sibling objects (e.g., `FundingAwardAmendment`, `FundingAwardRequirement`, `FundingDisbursement`, `FundingOpportunity`, `Budget`, etc.) and related custom objects (`GPSG_Risk_Assessment__c`, etc.).
- **Flows**: Retrieved `GPSG_Funding_Award_Congressional_Notifications` (via its API name `GPSG_Congressional_Notifications`), `Funding_Award_After_Update`, and 31 other GPSG-related flows.
- **Approval Process**: Extracted `FundingAward.Agreement_Approval`.
- **Apex & UI**: Extracted Apex classes (`GPSGPermissionSetUserRetrieval`, `GPSG_Application_Reviews`, etc.) and LWC components (including `docGenerationSampleFndSingleDocxLwcEnglish`).
- **DocGen Pathway**: Investigated `docGenerationSample` / `fndSingleDocxLwc`. The org uses the OmniStudio Foundation Document Generation framework, as evidenced by the `omnistudio__fndDocGenerationSampleLwc` and `omnistudio__docgenutil` static resources, and the `docGenerationSampleFndSingleDocxLwcEnglish` component.
- **Permissions & Layouts**: Extracted layouts, FlexiPages, and permission sets referencing the Award object.

**What was missing / unexpected:**
- No custom `EmailTemplate`s referencing "Award" or "GPSG" were found, and none were explicitly referenced in the `Agreement_Approval` approval process. The "Send Via Email Action" might be using standard UI or a different naming convention.
- OmniStudio `DocumentTemplate` or `OmniProcess` metadata types were not available/found, confirming this is relying on the OmniStudio Foundation / core static resources rather than custom OmniScripts.
