---
feature_area: grantmaking-pss
status: duplicate-of-catalog
catalog_id: P16
note: "Already cataloged as P16 (Grantmaking/PSS feature reference). Kept here for local browsing — do not re-catalog."
---

# Grantmaking & Public Sector Solutions — Feature Reference Guide

This guide consolidates official Salesforce documentation covering four key features used in the Post-Award / Grantmaking build. Use it as context when implementing related functionality.

> Source articles:
> - [Budget Management](https://help.salesforce.com/s/articleView?id=ind.grmk_budget_management_for_grantmaking.htm&type=5)
> - [Automatically Create Recurring Funding Award Requirements](https://help.salesforce.com/s/articleView?id=ind.gmrk_create_recurring_funding_award_requirements.htm&type=5)
> - [Compliant Data Sharing in Public Sector Solutions](https://help.salesforce.com/s/articleView?id=ind.psc_compliant_data_sharing.htm&type=5)
> - [Form Framework](https://help.salesforce.com/s/articleView?id=ind.psc_form_framework.htm&type=5)

---

## 1. Budget Management (Grantmaking)

Manage and track complex budgets broken down by category and period. Allocate actual budget expenditures at a high level or at the category and period level.

### Required Editions & Permissions

- Available in: **Lightning Experience**
- **Nonprofit Cloud for Grantmaking:** Enterprise, Unlimited, Developer
- **Public Sector Solutions:** Enterprise, Performance, Unlimited, Developer

| Action | Permission Required |
| --- | --- |
| Create and edit budgets | Read and Edit access on budget management objects |
| Use budgets in Experience Cloud | Grantmaking for Experience Cloud permission set AND (in orgs with Budgets set to private) Read access to any budget records used as a template |

### Overview

Budget management starts with creating a budget and its related budget periods and categories. After money is spent, track expenditures with budget allocations associated with each category value.

> **Important — Data Skew:** If you manage budgets with many category values, archive budgets that are not actively in use to avoid data skew. The number of category values is tied to the number of budgets and how many categories/periods are associated with each. Examples of large counts: 250 budgets × 170 categories × 5 periods, or 1,000 budgets × 17 categories × 3 periods.

### 1.1 Create Budget, Category, and Period Records

Budgets include start/end dates, amount or non-monetary quantity, the categories to budget for, and the time periods for the budget cycle.

1. From the App Launcher, find and select **Budget**.
2. Create a budget record, give it a name and other details, and save.
3. On the budget record, in the **Budget Category** related list, click **New**.
4. Enter category details. Enter a **Sequence** number to control display order (categories show in creation order if no sequence is entered).
5. Save. Click **Save & New** to add more categories.
6. On the budget record, in the **Budget Period** related list, click **New**.
7. Enter period details. Do **not** select **Submitted** (your admin uses that checkbox in Experience Cloud automation). Enter a **Sequence** number to control display order.
8. Save. Click **Save & New** to add more periods.

### 1.2 Enter a Planned Budget

Enter values for each budget period and category in a single grid form.

- Rows = categories (for example: Personnel, Supplies & Materials, Indirect Costs)
- Columns = periods (for example: Q1, Q2, Q3, Q4)

1. In the grid, enter the value for each category/period cell. Optionally enter a **Reason** for more context. Values save automatically as you move through the form.
2. When satisfied, **Submit** the budget. A submitted budget cannot be edited.

### 1.3 Track Actual Budget Spent — Experience Cloud Site

1. Open the application or budget record where you want to report actual spending.
2. In the grid, enter the actual amount for each category/period cell. Values save automatically; the **Variance** (Actual minus Planned) updates in real time. A negative variance means you have remaining budget.
3. If your admin enabled the automation for it, click **Submit**.

### 1.4 Track Actual Budget Spent — CRM (Budget Allocations)

Use budget allocations to track expenditures. Allocations can relate to a budget or to a budget category value.

1. From a budget or budget category value, in the **Budget Allocations** related list, click **New**.
2. Enter allocation details.
3. Save. Click **Save & New** to add more allocations.

### See Also

- [Grantmaking and Budget Management Data Model (Developer Guide)](https://developer.salesforce.com/docs/atlas.en-us.nonprofit_cloud_for_grantmaking_dev_guide.meta/nonprofit_cloud_for_grantmaking_dev_guide/npc_grant_budget_data_model.htm)
- [Add the Budget Component to the Budget Page Layout](https://help.salesforce.com/apex/HTViewHelpDoc?id=ind.grmk_add_budget_component_to_page_layout.htm)
- [Add Budget Components and Fields to the Application Form Page Layout](https://help.salesforce.com/apex/HTViewHelpDoc?id=ind.grmk_add_grantmaking_to_application_layout.htm)

---

## 2. Automatically Create Recurring Funding Award Requirements

Reduce manual effort and errors by automatically creating recurring funding award requirements from the Funding Award page.

### Required Editions & Permissions

- Available in: **Lightning Experience**
- Available in: Nonprofit Cloud for Grantmaking and Public Sector Solutions

| Action | Permission Required |
| --- | --- |
| Create and edit funding awards and related records | Grantmaking Manager permission set |
| Use funding awards in Experience Cloud | Grantmaking for Experience Cloud permission set |

### 2.1 Set Up Recurring Funding Award Requirements Automation

**Prerequisite:** Upgrade to Dynamic Actions in Lightning App Builder.

1. **Setup → Process Automation → Flows**. Select the **Create Recurring Funding Award Requirements** flow.
2. Customize the **Preview Funding Award Requirements** Data Table screen element:
   - Keep the default record collection `FundingAwardRequirementsRecords` or select a different one.
   - Set the **table edit mode** and **row selection method**.
     > Note: Even if Table Edit Mode is set to Edit, users without edit access on the record cannot save at end of flow.
   - Set minimum/maximum rows (leave blank for unlimited).
   - Under **Configure Columns**, set or reorder column headers (for example, Funding Award ID and Scheduled Date).
   - Under **Advanced**, keep or change the default output values for the Record Collection and Selected Rows Collection.
3. **Save as a new flow** and **activate** it.
4. **Object Manager → Funding Award object → Buttons, Links, and Actions → New Action**:
   - **Action Type:** Flow
   - **Flow:** select your customized flow
   - **Standard Label Type:** None
   - **Label:** `Create Recurring Requirements` (Name auto-populates)
   - Save.
5. Go to **Lightning Record Pages** (or open a funding award page and click **Edit Page** in Lightning App Builder).
6. Add the **Create Recurring Requirements** action to the **Highlights Panel**.
7. Save.

The **Create Recurring Requirements** button is now available on Funding Award pages.

### 2.2 Create Recurring Funding Award Requirements (End-User Workflow)

1. On a funding award page, click **Create Recurring Requirements**. Previously created requirements for this award are shown in the Funding Award Requirements section.
2. Enter requirement schedule details and click **Next**. A preview table of the requirements is displayed.
3. To edit a record, place your cursor in the cell and click the edit icon.
4. To add or remove records, use **Add Row** or **Remove Row** from the dropdown next to each row.
5. Click **Create Requirements**. The requirements are created and shown in a table.
6. Click **Finish**.

New requirement records appear on the funding award's **Related** tab and on the **Funding Award Requirements** page.

---

## 3. Compliant Data Sharing in Public Sector Solutions

Increase collaboration between internal and external users while maintaining compliance with regulations and policies for handling confidential or sensitive information.

### Required Editions & Permissions

- [View supported product editions](https://help.salesforce.com/apex/HTViewHelpDoc?id=ind.psc_supported_editions.htm)

| Action | Permission Required |
| --- | --- |
| Grant compliant data sharing access to internal users | Configure Compliant Data Sharing |
| Add/update participants, assign participant roles, manage participant status | Use Compliant Data Sharing |

### Overview

Use compliant data sharing to extend access to specific records that would otherwise be private — without custom code. For internal users, define participant roles for each parent entity and configure the default access level for each participant. For Experience Cloud users, enable collaboration on certain records. Assign or remove access for specific participant groups and roles as needed, and track participant access for compliance.

**Example use cases:**
- Give grantmakers access to manage budget, funding award, and individual application records, and share these records with other internal participants based on their role.
- Give grantseekers the ability to add participants to a grant application so they can update details such as the requested amount.

### 3.1 Supported Objects

Compliant Data Sharing can manage access to records for these objects in Public Sector Solutions:

| Object |
| --- |
| Account |
| Application Form |
| Application Form Evaluation |
| Budget |
| Case Proceeding |
| Custom object |
| Funding Award |
| Funding Opportunity |
| Individual Application |
| Interaction |
| Interaction Summary |
| Opportunity |
| Public Complaint |
| Recruitment Requisition |

### 3.2 Case Proceeding & Public Complaint — Special Handling

When you enable Compliant Data Sharing for **Case Proceeding** and **Public Complaint** objects, the **Case Proceeding Participant** and **Complaint Participant** objects can store two types of records:

1. Records that reference an **account or contact** — to associate a party with a case proceeding or public complaint.
2. Records that reference a **user or group** — to share a case proceeding or public complaint with them in a compliant manner.

For these objects, create **record types** and **page layouts** for both the party-association and data-sharing scenarios. Then, give users access to the record types and page layouts they need for their workflows.

### See Also

- [Compliant Data Sharing (Financial Services Cloud)](https://help.salesforce.com/s/articleView?id=ind.fsc_admin_cds.htm&type=5)
- [Create Record Types](https://help.salesforce.com/s/articleView?id=platform.creating_record_types.htm&type=5)
- [Create Page Layouts](https://help.salesforce.com/s/articleView?id=platform.customize_layoutcreate.htm&type=5)

---

## 4. Form Framework (Grantmaking & Public Sector Solutions)

Create complex applications and forms using your product's objects together with Omnistudio components or Flows. Use Form Framework to manage forms and applications in Salesforce CRM or through an Experience Cloud site.

### Required Editions

- Available in: **Lightning Experience**
- Available in: Nonprofit Cloud for Grantmaking and Public Sector Solutions ([view edition availability](https://help.salesforce.com/s/articleView?id=ind.grmk_form_framework_editions_permissions.htm&type=5))

### Capabilities

| Capability | Description |
| --- | --- |
| Multisection / multiphase applications | Applicants complete applications in phases. |
| Reusable sections | Reuse sections across forms for consistency. |
| Ad hoc sections | Add sections to in-progress or submitted applications. |
| Related record views | View reviews, decisions, and funding awards on related records. |
| Reviewer workspaces | Set up workspaces for reviewers to evaluate applications and make decisions on one screen. |
| Progress report forms | Provide progress report forms to grant recipients, and review/approve reports in a single view. |

### Use by Product

| Product | Use Case |
| --- | --- |
| **Grantmaking** | Simplify the funding application process with multiphase or multisection applications. |
| **Talent Recruitment Management** | Make the job application process more flexible and organized by adding sections to job applications. |

### See Also

- [Form Framework (Grantmaking)](https://help.salesforce.com/s/articleView?id=ind.grmk_form_framework.htm&type=5)
- [Grantmaking Overview](https://help.salesforce.com/s/articleView?id=ind.grmk_grantmaking.htm&type=5)
- [Multi-Section Job Applications (PSC)](https://help.salesforce.com/s/articleView?id=ind.psc_recruitment_multi_section_applications.htm&type=5)
