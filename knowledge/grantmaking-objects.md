---
feature_area: grantmaking-pss
status: duplicate-of-catalog
catalog_id: P7
catalog_ref: patterns/canonical/grantmaking-object-reference.md
note: "This is the literal source file for P7 (Platform data-model reference library). This copy is fuller (251 lines) than the current P7 canonical pointer file — flagged as an action item to upgrade P7 to embed this content directly. Do not re-catalog as a new asset."
---

# Salesforce Grantmaking - Objects & Field Metadata Reference

> **Source**: Salesforce Public Sector Solutions Developer Guide (Summer '26 / API v67.0)
> **License Required**: Grantmaking license must be enabled; `Manage Funding Awards` or `Manage Application` system permission required per object.

---

## Object Index

| Object | API Name | Available Since | Description |
| :--- | :--- | :--- | :--- |
| [Application Decision](#applicationdecision) | `ApplicationDecision` | API v56.0 | Final decision on an application (Award / Deny) |
| [Application Render Method](#applicationrendermethod) | `ApplicationRenderMethod` | Grantmaking API v61.0 | How a part of an application is rendered |
| [Application Review](#applicationreview) | `ApplicationReview` | API v56.0 | A review performed against an Individual Application |
| [Application Stage Definition](#applicationstagdefinition) | `ApplicationStageDefinition` | Grantmaking API v61.0 | A stage of an application |
| [Application Timeline](#applicationtimeline) | `ApplicationTimeline` | API v57.0 | Milestone dates in the application process |
| [Budget](#budget) | `Budget` | API v53.0 | Estimate of future revenue or expenses for a time period |
| [Budget Allocation](#budgetallocation) | `BudgetAllocation` | API v53.0 | Subsection of a Budget showing allocated resources |
| [Budget Category](#budgetcategory) | `BudgetCategory` | API v57.0 | Purpose of a budget line item |
| [Budget Category Value](#budgetcategoryvalue) | `BudgetCategoryValue` | API v57.0 | Budget values for a category and time period |
| [Budget Participant](#budgetparticipant) | `BudgetParticipant` | API v59.0 | User or group with access to a budget |
| [Budget Period](#budgetperiod) | `BudgetPeriod` | API v57.0 | Distinct time interval for a budget estimate |
| [Funding Award](#fundingaward) | `FundingAward` | API v57.0 | Award given to an individual or organization |
| [Funding Award Amendment](#fundingawardamendment) | `FundingAwardAmendment` | API v57.0 | Modification to scope or finances of an approved award |
| [Funding Award Participant](#fundingawardparticipant) | `FundingAwardParticipant` | API v59.0 | User or group with access to a funding award |
| [Funding Award Requirement](#fundingawardrequirement) | `FundingAwardRequirement` | API v57.0 | Deliverable or milestone for an award or disbursement |
| [Funding Award Rqmt Section](#fundingawardrqmtsection) | `FundingAwardRqmtSection` | API v62.0 | Part of a requirement to be completed or reviewed |
| [Funding Disbursement](#fundingdisbursement) | `FundingDisbursement` | API v57.0 | Payment made or scheduled to a funding recipient |
| [Funding Opportunity](#fundingopportunity) | `FundingOpportunity` | API v57.0 | Pool of money available for a specific purpose |
| [Funding Opp Participant](#fundingoppparticipant) | `FundingOppParticipant` | API v60.0 | User or group with access to a funding opportunity |
| [Indicator Assignment](#indicatorassignment) | `IndicatorAssignment` | API v59.0 | Assignment of an indicator to measure outcome performance |
| [Indicator Performance Period](#indicatorperformanceperiod) | `IndicatorPerformancePeriod` | API v59.0 | Time period and baseline for indicator results |
| [Individual Application](#individualapplication) | `IndividualApplication` | API v50.0 | Application form submitted by an individual or organization |
| [Individual Application Task](#individualapplicationtask) | `IndividualApplicationTask` | Grantmaking API v61.0 | Task related to an application |
| [Indv Application Task Participant](#indvapplicationtaskparticipant) | `IndvApplicationTaskParticipant` | API v61.0 | User or group with access to an application task |
| [Individual Appln Participant](#individualapplnparticipant) | `IndividualApplnParticipant` | API v59.0 | User or group with access to an individual application |
| [Outcome Activity](#outcomeactivity) | `OutcomeActivity` | API v59.0 | Junction between an outcome and a related activity object |
| [Preliminary Application Ref](#preliminaryapplicationref) | `PreliminaryApplicationRef` | API v49.0 | Saved applications and pre-screening forms |
| [Program](#program) | `Program` | API v57.0 | Enrollment and disbursement of benefits in a program |

---

## ApplicationDecision

Represents a final decision performed for the specified Application. Available in API version 56.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Application` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Application Decision | `ApplicationDecision` | Picklist | Create, Filter, Group, Nillable, Sort, Update | Decision outcome: `Award`, `Deny` |
| Application | `ApplicationId` | Reference → `IndividualApplication` | Create, Filter, Group, Sort, Update | The application this decision applies to |
| Comment | `Comment` | Textarea | Create, Nillable, Update | Decision commentary |
| Decision Authority | `DecisionAuthorityId` | Reference → `User` | Create, Filter, Group, Nillable, Sort, Update | User responsible for the decision |
| Name | `Name` | String (Autonumber) | Defaulted on create, Filter, idLookup, Sort | Auto-generated name |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Preliminary Application Ref | `PreliminaryApplicationRefId` | Reference → `PreliminaryApplicationRef` | Create, Filter, Group, Nillable, Sort, Update | Linked pre-screening form |

---

## ApplicationRenderMethod

Represents how a part of an application can be rendered. Available in Grantmaking API version 61.0 and later.

*Field metadata not yet published in public API reference.*

---

## ApplicationReview

Represents a review performed against a specified Individual Application. Available in API version 56.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Application` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Application | `ApplicationId` | Reference → `IndividualApplication` | Create, Filter, Group, Sort, Update | The application under review |
| Application Category | `ApplicationCategory` | Picklist | Filter, Group, Nillable, Sort | `Basic`, `Special` |
| Application Recommendation | `ApplicationRecommendation` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Ask for Revisions`, `Award`, `Deny` |
| Assigned Date | `AssignedDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Date application was assigned to reviewer |
| Assigned User | `AssignedUserId` | Reference → `User` | Create, Filter, Group, Nillable, Sort, Update | User assigned to perform the review |
| Comment | `Comment` | Textarea | Create, Nillable, Update | Review notes |
| Condition | `Condition` | Textarea | Create, Nillable, Update | Conditions applicable to the applicant |
| Due Date | `DueDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Deadline for completing the review |
| End Date | `EndDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Date the review was completed |
| Is Submitted | `IsSubmitted` | Boolean | Create, Defaulted on create, Filter, Group, Sort, Update | Whether review has been submitted (default: false) |
| Name | `Name` | String (Autonumber) | Defaulted on create, Filter, idLookup, Sort | Auto-generated name |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Reviewed By | `ReviewedById` | Reference → `User` | Create, Filter, Group, Nillable, Sort, Update | Person who reviewed the application |
| Start Date | `StartDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Date review started |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Not Started`, `In Progress`, `Completed`, `Cancelled` |
| Submission Date | `SubmissionDate` | Date | Filter, Group, Nillable, Sort | Date applicant submitted the application |
| Title | `Title` | String | Create, Filter, Group, Nillable, Sort, Update | Detailed name of the preliminary application reference |

---

## ApplicationStageDefinition

Represents a stage of an application. Available in Grantmaking API version 61.0 and later.

*Field metadata not yet published in public API reference.*

---

## ApplicationTimeline

Represents the milestone dates in the application process. Available in API version 57.0 and later.

*Field metadata not yet published in public API reference.*

---

## Budget

Tracks an estimate of future revenue or expenses during a specific time period. Available in API version 53.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grants Management or Grantmaking license enabled + `Manage Budgets` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Account | `AccountId` | Reference → `Account` | Create, Filter, Group, Nillable, Sort, Update | Associated account (v58.0+) |
| Amount | `Amount` | Currency | Create, Filter, Nillable, Sort, Update | Total budget funds |
| Description | `Description` | String | Create, Filter, Group, Nillable, Sort, Update | Description of the budget |
| Estimated Utilization Amount | `EstimatedUtilizationAmount` | Currency | Create, Filter, Nillable, Sort, Update | Estimated amount to be utilized |
| Is Submitted | `IsSubmitted` | Boolean | Create, Defaulted on create, Filter, Group, Sort, Update | Whether budget has been submitted (default: false; v58.0+) |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Budget name |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Parent Budget | `ParentBudgetId` | Reference → `Budget` | Create, Filter, Group, Nillable, Sort, Update | Parent budget (v56.0+) |
| Period End Date | `PeriodEndDate` | Date | Create, Filter, Group, Nillable, Sort, Update | End of the date range |
| Period Name | `PeriodName` | String | Create, Filter, Group, Nillable, Sort, Update | Name of the time period |
| Period Start Date | `PeriodStartDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Start of the date range |
| Program | `ProgramId` | Reference → `Program` | Create, Filter, Group, Nillable, Sort, Update | Associated program (v58.0+) |
| Quantity | `Quantity` | Double | Create, Filter, Nillable, Sort, Update | Non-currency budget quantity (e.g., hours, resources) |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Active`, `Archived`, `Planned` |
| Type | `Type` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Department`, `Program`, `Project` |
| Utilized Amount | `UtilizedAmount` | Currency | Create, Filter, Nillable, Sort, Update | Amount already utilized from the budget |

---

## BudgetAllocation

Represents a subsection of a Budget that shows where allocated resources are being applied. Available in API version 53.0 and later.

*Field metadata not yet published in public API reference.*

---

## BudgetCategory

Represents the purpose of the budget line item. Available in API version 57.0 and later.

*Field metadata not yet published in public API reference.*

---

## BudgetCategoryValue

Captures budget values for category and time period. Available in API version 57.0 and later.

*Field metadata not yet published in public API reference.*

---

## BudgetParticipant

Represents information about a user or group of participants who have access to a budget. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## BudgetPeriod

Defines a distinct time interval in which the estimate applies. Available in API version 57.0 and later.

*Field metadata not yet published in public API reference.*

---

## FundingAward

Represents an award given to an individual or organization to facilitate a goal related to the funder's mission. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Funding Awards` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Amount | `Amount` | Currency | Create, Filter, Nillable, Sort, Update | Total award amount |
| Award Number | `AwardNumber` | String (Autonumber) | Defaulted on create, Filter, Sort | Unique identifier; auto-generated |
| Awardee | `AwardeeId` | Reference → `Account` | Create, Filter, Group, Nillable, Sort, Update | Organization or individual receiving funding |
| Budget | `BudgetId` | Reference → `Budget` | Create, Filter, Group, Nillable, Sort, Update | Budget used to track award funding |
| Contact | `ContactId` | Reference → `Contact` | Create, Filter, Group, Nillable, Sort, Update | Individual receiving the award |
| Decision Date | `DecisionDate` | DateTime | Create, Filter, Nillable, Sort, Update | Date and time of the award decision |
| End Date | `EndDate` | DateTime | Create, Filter, Nillable, Sort, Update | Contract end date |
| Funding Opportunity | `FundingOpportunityId` | Reference → `FundingOpportunity` | Create, Filter, Group, Nillable, Sort, Update | Associated funding opportunity |
| Individual Application | `IndividualApplicationId` | Reference → `IndividualApplication` | Create, Filter, Group, Nillable, Sort, Update | Related individual application |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Name of the funding award |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Parent Funding Award | `ParentFundingAwardId` | Reference → `FundingAward` | Create, Filter, Group, Nillable, Sort, Update | Parent award for amendments |
| Start Date | `StartDate` | DateTime | Create, Filter, Nillable, Sort, Update | Contract start date |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Active`, `Cancelled`, `Completed` |

---

## FundingAwardAmendment

Represents a modification to the scope or finances of a previously approved award. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Funding Awards` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Adjusted Award Amount | `AdjustedAwardAmount` | Currency | Create, Filter, Nillable, Sort, Update | Actual approved adjustment amount |
| Adjusted End Date | `AdjustedEndDate` | DateTime | Create, Filter, Nillable, Sort, Update | Actual adjusted end date |
| Approval Status | `ApprovalStatus` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Approved`, `In Review`, `New`, `Rejected` |
| Comments | `Comments` | Textarea | Create, Nillable, Update | Notes on approval or rejection |
| Funding Award | `FundingAwardId` | Reference → `FundingAward` | Create, Filter, Group, Sort | Parent award being amended |
| Name | `Name` | String (Autonumber) | Defaulted on create, Filter, idLookup, Sort | Auto-generated amendment name |
| Proposed Award Amount | `ProposedAwardAmount` | Currency | Create, Filter, Nillable, Sort, Update | Requested adjustment to award amount |
| Proposed End Date | `ProposedEndDate` | DateTime | Create, Filter, Nillable, Sort, Update | Requested change to end date |
| Reason | `Reason` | Textarea | Create, Nillable, Update | Reason for the amendment request |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Approved`, `Draft`, `Rejected`, `Submitted` |
| Type | `Type` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Administrative`, `Amount`, `Scope`, `Timeline` |

---

## FundingAwardParticipant

Represents information about a user or group of participants who have access to a funding award. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## FundingAwardRequirement

Represents a deliverable or milestone for a funding award or funding disbursement. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Funding Awards` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Approval Status | `ApprovalStatus` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Approved`, `In Review`, `New`, `Rejected` |
| Assigned Contact | `AssignedContactId` | Reference → `Contact` | Create, Filter, Group, Nillable, Sort, Update | Person responsible for submitting the requirement |
| Assigned User | `AssignedUserId` | Reference → `User` | Create, Filter, Group, Nillable, Sort, Update | User who submits the requirement |
| Description | `Description` | Textarea | Create, Nillable, Update | Description of the requirement |
| Due Date | `DueDate` | DateTime | Create, Filter, Nillable, Sort, Update | Submission deadline |
| Funding Award | `FundingAwardId` | Reference → `FundingAward` | Create, Filter, Group, Sort | Associated funding award |
| Funding Disbursement | `FundingDisbursementId` | Reference → `FundingDisbursement` | Create, Filter, Group, Nillable, Sort, Update | Disbursement gated on this requirement |
| Is Approved | `IsApproved` | Boolean | Defaulted on create, Filter, Group, Sort | Whether submitted info matches requirement (default: false) |
| Is Read Only External Access | `IsReadOnlyExternalAccess` | Boolean | Defaulted on create, Filter, Group, Sort | Whether requirement is submitted (default: false) |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Requirement name |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Approved`, `Delayed`, `In Progress`, `Open`, `Rejected`, `Submitted` |
| Submitted Date | `SubmittedDate` | DateTime | Create, Filter, Nillable, Sort, Update | Actual submission date |
| Type | `Type` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Combined Report`, `Contract`, `Financial Report`, `Narrative Report` |

---

## FundingAwardRqmtSection

Represents a part of a funding award requirement to be completed or reviewed. Available in API version 62.0 and later.

*Field metadata not yet published in public API reference.*

---

## FundingDisbursement

Represents a payment that has been made or scheduled to be made to a funding recipient. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Funding Awards` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Amount | `Amount` | Currency | Create, Filter, Nillable, Sort, Update | Total amount disbursed to the awardee |
| Disbursement Date | `DisbursementDate` | DateTime | Create, Filter, Nillable, Sort, Update | Actual date and time funds were disbursed |
| Funding Award | `FundingAwardId` | Reference → `FundingAward` | Create, Filter, Group, Sort | Associated funding award |
| Is Approved | `IsApproved` | Boolean | Defaulted on create, Filter, Group, Sort | Whether disbursement is approved (default: false) |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Disbursement name |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Payment Method Type | `PaymentMethodType` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Cash`, `Check`, `EFT`, `Wire` |
| Payment Number | `PaymentNumber` | String (Autonumber) | Defaulted on create, Filter, Sort | Unique payment identifier |
| Scheduled Date | `ScheduledDate` | DateTime | Create, Filter, Nillable, Sort, Update | Scheduled disbursement date and time |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Approved`, `Cancelled`, `Paid`, `Pending Approval`, `Processing`, `Returned`, `Scheduled` |

---

## FundingOpportunity

The pool of money available for distribution for a specific purpose. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

**Access**: Grantmaking license enabled + `Manage Funding Awards` permission.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Action Plan Template | `ActionPlanTemplateId` | Reference → `ActionPlanTemplate` | Create, Filter, Group, Nillable, Sort, Update | Template representing application sections |
| Application Instructions | `ApplicationInstructions` | Textarea | Create, Nillable, Update | Instructions for applicants |
| Application Timeline | `ApplicationTimelineId` | Reference → `ApplicationTimeline` | Create, Filter, Group, Nillable, Sort, Update | Milestone timeline for the application process |
| Budget Template | `BudgetTemplateId` | Reference → `Budget` | Create, Filter, Group, Nillable, Sort, Update | Budget template for applicants |
| Description | `Description` | Textarea | Create, Nillable, Update | Opportunity description and requirements |
| End Date | `EndDate` | DateTime | Create, Filter, Nillable, Sort, Update | Date acceptance of applications ended |
| Maximum Funding Amount | `MaximumFundingAmount` | Currency | Create, Filter, Nillable, Sort, Update | Maximum award amount |
| Minimum Funding Amount | `MinimumFundingAmount` | Currency | Create, Filter, Nillable, Sort, Update | Minimum award amount |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Opportunity name |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Parent Funding Opportunity | `ParentFundingOpportunityId` | Reference → `FundingOpportunity` | Create, Filter, Group, Nillable, Sort, Update | Parent opportunity (v59.0+) |
| Program | `ProgramId` | Reference → `Program` | Create, Filter, Group, Nillable, Sort, Update | Associated program (v58.0+) |
| Start Date | `StartDate` | DateTime | Create, Filter, Nillable, Sort, Update | Date acceptance of applications started |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Active`, `Cancelled`, `Completed`, `Planned` |

---

## FundingOppParticipant

Represents information about a user or group of participants who have access to a funding opportunity. Available in API version 60.0 and later.

*Field metadata not yet published in public API reference.*

---

## IndicatorAssignment

Represents the assignment of an indicator definition used to measure the performance of an outcome or a related activity. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## IndicatorPerformancePeriod

Represents information about a specified time period including the frequency at which indicator results should be calculated and the baseline value of the indicator. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## IndividualApplication

Represents an application form submitted by an individual or organization. Available in API version 50.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Account | `AccountId` | Reference → `Account` | Create, Filter, Group, Nillable, Sort, Update | Applicant's account |
| Application Case | `ApplicationCaseId` | Reference → `Case` | Create, Filter, Group, Nillable, Sort, Update | Case related to this application |
| Application Change Overview | `ApplicationChangeOverview` | Textarea | Create, Nillable, Update | Einstein-generated overview of changes between versions (v62.0+) |
| Application Name | `ApplicationName` | String | Create, Filter, Group, Nillable, Sort, Update | Descriptive name (Grantmaking/PSS; v57.0+) |
| Application Overview | `ApplicationOverview` | Textarea | Create, Nillable, Update | Einstein-generated historical overview (v62.0+) |
| Application Reference Number | `ApplicationReferenceNumber` | String | Create, Filter, Group, Nillable, Sort, Update | Custom reference number |
| Application Type | `ApplicationType` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Change Of Circumstance`, `New`, `Recertification`, `Renewal` |
| Applied Date | `AppliedDate` | DateTime | Create, Filter, Sort, Update | Date application was received |
| Approved Date | `ApprovedDate` | DateTime | Create, Filter, Nillable, Sort, Update | Date application was approved |
| Budget | `BudgetId` | Reference → `Budget` | Create, Filter, Group, Nillable, Sort, Update | Associated budget (Grantmaking/PSS; v57.0+) |
| Category | `Category` | Picklist | Create, Filter, Group, Sort, Update | `License`, `Permit`, `Grant Application`, `Letter of Intent` |
| Contact | `ContactId` | Reference → `Contact` | Create, Filter, Group, Nillable, Sort, Update | Associated contact |
| Description | `Description` | String | Create, Filter, Group, Nillable, Sort, Update | Applicant-provided description |
| Funding Opportunity | `FundingOpportunityId` | Reference → `FundingOpportunity` | Create, Filter, Group, Nillable, Sort, Update | Associated funding opportunity (v57.0+) |
| Funding Request Purpose | `FundingRequestPurpose` | Textarea | Create, Nillable, Update | Purpose of the requested funds (v57.0+) |
| Internal Status | `InternalStatus` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Invited`, `In Progress`, `Submitted`, `Application Accepted`, `Revision Requested`, `In Review`, `Approved`, `Denied` |
| Is Owner Editable | `IsOwnerEditable` | Boolean | Create, Defaulted on create, Filter, Group, Sort, Update | Whether owner ID can be changed (default: false) |
| Is Submitted | `IsSubmitted` | Boolean | Create, Defaulted on create, Filter, Group, Sort, Update | Whether application has been submitted (v58.0+; default: false) |
| Name | `Name` | String (Autonumber) | Defaulted on create, Filter, idLookup, Sort | Auto-generated unique ID |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Record Type | `RecordTypeId` | Reference → `RecordType` | Create, Filter, Group, Nillable, Sort, Update | Associated record type |
| Requested Amount | `RequestedAmount` | Currency | Create, Filter, Nillable, Sort, Update | Amount requested (v57.0+) |
| Requirements Complete Date | `RequirementsCompleteDate` | DateTime | Create, Filter, Nillable, Sort, Update | Date when all requirements were fulfilled |
| Saved Application Ref | `SavedApplicationRefId` | Reference → `PreliminaryApplicationRef` | Create, Filter, Group, Nillable, Sort, Update | Linked saved application reference |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Invited`, `In Progress`, `Submitted`, `Application Accepted`, `Revision Requested`, `In Review`, `Approved`, `Denied` |
| Was Returned | `WasReturned` | Boolean | Create, Defaulted on create, Filter, Group, Sort, Update | Whether application was returned to applicant (default: false) |

---

## IndividualApplicationTask

Represents a task related to an application. Available in Grantmaking API version 61.0 and later.

*Field metadata not yet published in public API reference.*

---

## IndvApplicationTaskParticipant

Represents information about a user or group of participants who have read or write access to an individual application task. Available in API version 61.0 and later.

*Field metadata not yet published in public API reference.*

---

## IndividualApplnParticipant

Represents information about a user or group of participants who have access to an individual application. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## OutcomeActivity

Represents a junction between an outcome and the object that's related to the activity undertaken by an organization to achieve that outcome. Available in API version 59.0 and later.

*Field metadata not yet published in public API reference.*

---

## PreliminaryApplicationRef

Represents the saved applications and pre-screening forms. Available in API version 49.0 and later.

*Field metadata not yet published in public API reference.*

---

## Program

Represents information about the enrollment and disbursement of benefits in a program. Available in API version 57.0 and later.

**Supported Calls**: `create()`, `delete()`, `describeLayout()`, `describeSObjects()`, `getDeleted()`, `getUpdated()`, `query()`, `retrieve()`, `search()`, `undelete()`, `update()`, `upsert()`

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| Active Enrollee Count | `ActiveEnrolleeCount` | Double | Create, Filter, Nillable, Sort, Update | Count of enrollees in active status |
| Additional Context | `AdditionalContext` | Textarea | Create, Nillable, Update | Additional context about the program |
| Current Month Disbursement Count | `CurrentMonthDisbCount` | Double | Create, Filter, Nillable, Sort, Update | Benefits disbursed in current month |
| Current Year Disbursement Count | `CurrentYearDisbCount` | Double | Create, Filter, Nillable, Sort, Update | Benefits disbursed in current year |
| End Date | `EndDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Date the program ends |
| Name | `Name` | String | Create, Filter, Group, idLookup, Sort, Update | Program name (required) |
| Owner | `OwnerId` | Reference → `Group`, `User` | Create, Defaulted on create, Filter, Group, Sort, Update | Record owner |
| Parent Program | `ParentProgramId` | Reference → `Program` | Create, Filter, Group, Nillable, Sort, Update | Associated parent program (v59.0+) |
| Previous Month Disbursement Count | `PreviousMonthDisbCount` | Double | Create, Filter, Nillable, Sort, Update | Benefits disbursed in previous month |
| Previous Year Disbursement Count | `PreviousYearDisbCount` | Double | Create, Filter, Nillable, Sort, Update | Benefits disbursed in previous year |
| Start Date | `StartDate` | Date | Create, Filter, Group, Nillable, Sort, Update | Date the program begins |
| Status | `Status` | Picklist | Create, Filter, Group, Nillable, Sort, Update | `Active`, `Cancelled`, `Completed`, `Planned` |
| Summary | `Summary` | String | Create, Filter, Group, Nillable, Sort, Update | Summary of the program |
| Total Enrollee Count | `TotalEnrolleeCount` | Double | Create, Filter, Nillable, Sort, Update | Total count of program enrollees |
| Usage Type | `UsageType` | Picklist (Restricted) | Create, Filter, Group, Nillable, Sort, Update | `ProgramManagement` |

---

## References

- [Salesforce Public Sector Solutions Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm)
- [Grantmaking Data Model Diagram](https://developer.salesforce.com/docs/platform/data-models/guide/grantmaking.html)
- [Grantmaking Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.nonprofit_cloud_for_grantmaking_dev_guide.meta/nonprofit_cloud_for_grantmaking_dev_guide/npc_introduction_to_nonprofit_cloud_for_grantmaking.htm)
