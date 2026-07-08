---
feature_area: salesforce-flow
status: duplicate-of-catalog
catalog_id: P14
note: "Already cataloged as P14 (Flow Approval Processes knowledge library). Kept here for local browsing — do not re-catalog."
---

# Flow Approval Processes — Implementation Guide

> **Audience:** Cursor AI agent building Flow Approval Process automation in Salesforce.
> **Sources:** Scraped directly from help.salesforce.com. All URLs are listed per section.
> **Metadata API Note:** The developer.salesforce.com Metadata API pages for `Flow` and `Orchestration` types are JavaScript-rendered and could not be scraped. Direct links: [Flow Metadata](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_visual_workflow.htm) | [Orchestration Metadata](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_orchestration.htm). The `FlowDefinition` metadata type with `processType` of `Orchestrator` or `ApprovalOrchestration` is the deployment artifact — deploy as a `.flow-meta.xml` file under `force-app/main/default/flows/`.

---

## Table of Contents

1. [Feature Overview](#1-feature-overview)
2. [Permissions & Editions](#2-permissions--editions)
3. [Key Concepts](#3-key-concepts)
   - [Flow Approval Process Types](#31-flow-approval-process-types)
   - [Building Blocks: Stages, Steps, Flows](#32-building-blocks-stages-steps-flows)
   - [Triggers](#33-triggers)
   - [Approval Submissions & Orchestration Runs](#34-approval-submissions--orchestration-runs)
   - [Record Locking](#35-record-locking)
   - [Versioning](#36-versioning)
4. [Approval Steps — Deep Reference](#4-approval-steps--deep-reference)
5. [Background Steps](#5-background-steps)
6. [Build a Flow Approval Process — Step-by-Step](#6-build-a-flow-approval-process--step-by-step)
7. [Deploy & Activate](#7-deploy--activate)
8. [Make It Available to Users](#8-make-it-available-to-users)
9. [Resources Reference](#9-resources-reference)
10. [Limits](#10-limits)
11. [Considerations & Pitfalls](#11-considerations--pitfalls)
12. [Troubleshooting & Debugging](#12-troubleshooting--debugging)
13. [Managing Submissions & Work Items](#13-managing-submissions--work-items)
14. [Migration from Legacy Approval Processes](#14-migration-from-legacy-approval-processes)
15. [Metadata Deployment Reference](#15-metadata-deployment-reference)

---

## 1. Feature Overview

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals.htm&type=5

Flow Approval Processes is a Salesforce automation feature for building multi-step, multi-user, multi-system approval workflows directly in Flow Builder. It replaces the legacy Approval Processes feature for complex scenarios.

**When to use Flow Approval Processes:**
- Record reviews requiring multilevel, multiuser, or multisystem approval.
- Need for logic/branching to handle multiple approval scenarios.
- Getting input from people inside **and** outside the company (Experience Cloud).
- Interacting with external systems during approval.

**Core mechanism:**
- A **flow approval process** is a sequence of **stages**, each containing one or more **steps**.
- **Approval steps** assign work items to users, groups, or queues.
- **Background steps** run automated actions with no user interaction.
- When a record is submitted, an **orchestration run** is created and manages the process to completion.
- Approvers act via the **Orchestration Work Guide** Lightning App Builder component on the record page, from the **Approval Work Items** list view, or by replying to email notifications.

---

## 2. Permissions & Editions

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_about.htm&type=5

**Available in:** Lightning Experience — Enterprise, Unlimited, Performance, Developer editions.

| Permission | Grants |
| --- | --- |
| `Manage Flow` | Open, edit, create, activate/deactivate a flow approval process in Flow Builder |
| `Approval Designer` | See the "Manage All Flow Approval Processes" tile in the Approvals app |
| `Run Flows` | Complete assigned approval work items in the Work Guide or from the Approval Work Items list view |
| `Configure Compliant Data Sharing` | (PSC/Grantmaking only) Grant CDS access |
| `View All Data` | Required in addition to `Manage Flow` to activate a **record-triggered** flow approval process |
| `Approval Admin` (system permission) | Recall any in-progress approval submission |

---

## 3. Key Concepts

### 3.1 Flow Approval Process Types

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts.htm&type=5

| Type | How It Starts | Created In |
| --- | --- | --- |
| **Autolaunched (No Trigger)** | A user manually submits a record via a button or the Request Approval component | Approvals app or Setup → Flows |
| **Record-Triggered** | Automatically starts after a record is created or updated and entry criteria are met | Setup → Flows |

> **Distribution note:** The type also determines how the flow approval process can be packaged and distributed.

---

### 3.2 Building Blocks: Stages, Steps, Flows

**Sources:**
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_building_blocks.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_stages.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_steps.htm&type=5

#### Stages

- **Stages run sequentially** — only one stage can be in progress at a time.
- A flow approval process must have **at least 1 stage**.
- Stages group related steps into logical phases.
- You cannot control when a stage *starts* (they run sequentially), but you control when a stage *completes* via exit conditions.

**Stage Exit Conditions:**

| Condition | Requires |
| --- | --- |
| When all steps have been marked Completed, the stage is marked Completed | — |
| When the specified requirements are met, the stage is marked Completed | Up to 10 requirements |
| When the specified evaluation flow returns True, the stage is marked Completed | An active evaluation flow |

**Stage Fault Paths:**
- Each stage can have its own fault path with Decision and Stage elements.
- Fault path stages can contain both approval and background steps.
- The final submission status with a configured fault path that ends without error is **Approved** (if no work items rejected) or **Rejected** (if any work item was rejected).
- A stage without a fault path that errors results in `Error` status.

**Stage Statuses:** `In Progress`, `Completed`, `Canceled`, `Discontinued`, `Error`

#### Steps

Steps are grouped inside stages and can run **sequentially**, **concurrently**, or based on requirements.

| Step Type | Description |
| --- | --- |
| **Approval Step** | Assigns work to a user/group/queue; requires user action; calls a **screen flow** |
| **Background Step** | Runs an **autolaunched flow** synchronously or asynchronously; no user interaction |

**Step Statuses:** `Not Started`, `In Progress`, `Completed`, `Discontinued`, `Error`

> The `Step` resource in Flow Approval Processes is **not** related to the discontinued `Step` element in regular flows.
> The `Stage` element in Flow Approval Processes is **not** related to the `Stage` resource in regular flows.

---

### 3.3 Triggers

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_trigger.htm&type=5

- Record-triggered flow approval processes fire **after a record is saved**.
- Trigger options: `A record is created`, `A record is updated`, `A record is created or updated`.
- **Cannot** configure a flow approval process to start after a record is deleted.
- Use **Flow Trigger Explorer** to order record-triggered flow approval processes alongside other orchestrations and flows.

---

### 3.4 Approval Submissions & Orchestration Runs

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_approval_submission.htm&type=5

**Approval Submission** — created each time a record is submitted for approval. Tracks the lifecycle of the review.

| Submission Status | Meaning |
| --- | --- |
| `In Progress` | Started |
| `Approved` | Approved |
| `Rejected` | Rejected |
| `Canceled` | Manually canceled |
| `Recalled` | Manually recalled |
| `Error` | Associated orchestration run failed |

**Comment storage:**

| When the comment is made | Stored on |
| --- | --- |
| Submitting an approval request | Approval Submission record |
| Reviewing an approval work item | Approval Work Item record |
| Canceling or recalling | Approval Submission detail record |

**Orchestration Run** — the run-time instance of the active flow approval process, linked to the approval submission.

| Run Status | Meaning |
| --- | --- |
| `In Progress` | Submission started |
| `Completed` | Submission approved or rejected |
| `Canceled` | Submission manually canceled or recalled |
| `Error` | Run, stage, step, or called flow encountered an error |

**Record Refresh:** Each time an orchestration run resumes, referenced record variables and record collection variables are refreshed with their latest values. `$Record_Prior` is **not** refreshed for record-triggered approval processes.

---

### 3.5 Record Locking

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concept_record_locking.htm&type=5

Record locking controls who can change a record associated with an in-progress approval submission. A record can have **only 1 lock at a time** — attempting to lock a locked record causes an error.

#### Consecutively Run Approval Steps
Use the **Lock the record** checkbox in the Approval Step properties panel (under *Select Where to Complete the Action*). The record unlocks automatically when the approver completes the step.

#### Concurrently Run Approval Steps
Do **not** use the step-level `Lock the record` checkbox. Manage locking via background steps or at the flow-approval-process level:

**Flow-level locking:**
1. Add a stage *before* any approval stages → background step calls an autolaunched flow with the [Lock Record](https://help.salesforce.com/apex/HTViewHelpDoc?id=platform.flow_ref_elements_actions_lockrecord.htm) action: `Action = Lock`, `Record ID = <record under review>`.
2. Add a stage *after* all approval stages → background step unlocks: `Action = Unlock`, `Record ID = <record under review>`.

**Stage-level locking:**
1. Add a background step at stage start: `Action = Lock`, `Record ID = <record under review>`. Configure each approval step to require this background step to be Completed before starting.
2. Add a second background step to unlock when all concurrent approval steps are Completed:
   - ≤10 approval steps: use `When the specified requirements are met` (one requirement per approval step status = Completed).
   - >10 approval steps: use an evaluation flow that checks all step statuses and sets `isOrchestrationConditionMet`.

**Allow approver to edit locked record:**
- Consecutive: enable `Allow approver to edit the locked record` in the Approval Step panel. Any assigned approver (not delegate) can edit.
- Concurrent: set the `Allowed ID` input of the locking background step to the user/group/queue/role allowed to edit.

---

### 3.6 Versioning

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts.htm&type=5

- Flow approval processes have two levels of versioning: the **flow approval process version** and the **version of each flow called** by the process.
- Only **one version** of a flow approval process can be active at a time.
- In-progress submissions continue running on the version they started with even after a newer version is activated.

---

## 4. Approval Steps — Deep Reference

**Sources:**
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_approval_step.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_build_create_step_to_interact_with_reviewers.htm&type=5

### Required Screen Flow Variables

Every screen flow called by an approval step **must** contain this variable:

| Variable Name | Type | Required | Valid Values | Purpose |
| --- | --- | --- | --- | --- |
| `approvalDecision` | Text (Output) | **Yes** | `Approve` or `Reject` | Stores the reviewer's decision. Used in downstream conditions. If missing or invalid, the flow approval process fails. |
| `approvalComments` | Text (Output) | No | Any text | Stored in the Comments field of the approval work item. |

> **Design tip:** An approver can approve/reject *without* running the screen flow. So the screen flow should only show what the approver needs to make a decision. Use a **background step after** the approval step to update records or trigger notifications.

### When to Start the Step

| Option | Requires |
| --- | --- |
| When the stage starts, the step starts | — |
| When another step is marked Completed, the step starts | Name of the prerequisite step |
| When the specified requirements are met, the step starts | Up to 10 requirements |
| When the specified evaluation flow returns True, the step starts | Name of an active evaluation flow |

### Assignment Types

| Type | Accepts |
| --- | --- |
| `User` | Internal user username or credentialed Experience Cloud site visitor username |
| `Group` | API name of a public group (internal users only, or credentialed Aura site visitors only, or credentialed LWR site visitors only) |
| `Queue` | API name of a queue (same purity constraints as Group) |
| `Resource` | API name of a variable resolved at run time to any of the above |

> **Important:** Do **not** use `$User` for the Resource type. In system context, `$User` evaluates to the Automated Process User — an approval step cannot be assigned to the system user.

### When to Complete the Step

| Option | Requires |
| --- | --- |
| When the assigned user has completed the screen flow, the step is marked Completed | — |
| When the specified requirements are met, the step is marked Completed | Up to 10 requirements |
| When the specified evaluation flow returns True, the step is marked Completed | Name of an active evaluation flow |

### Default Email Notifications

Flow approval processes send emails from the **Automated Process User Email Address** in Process Automation Settings. Fallback order: org-wide Default No-Reply Address → any verified org-wide email → running user email → no email (system context with no user).

| Recipient | Default Notification |
| --- | --- |
| Assigned internal user | Personalized email with link to internal record page |
| Assigned credentialed Experience Cloud visitor | Personalized email with link to record page of oldest live site they're a member of |
| Assigned group/queue of internal users | General email with link to internal record page |
| Assigned group/queue of site visitors — all on same site | General email with link to that site's record page |
| Assigned group/queue of site visitors — multiple sites in common | General email with link to oldest shared live site |
| Assigned group/queue — no site in common | General email with internal link (invalid for site visitors; they use Approval Work Items list in Experience Cloud) |
| Delegated internal user | Personalized email with link to internal record page AND link to approval work item |

### Email Response (Keyword Approval)

Enable **`Enable email approval response`** in Process Automation Settings to let approvers approve/reject by replying to their email notification with specified keywords.

When this setting is enabled, group/queue members each receive their **own** copy (To field) of the notification. When disabled, they receive a **BCC** copy.

### Step Access Requirements

- Internal approvers need access to the **internal Lightning record page** for the record under review.
- Experience Cloud site visitors need access to the **object detail page** in an Aura or LWR site.
- To complete steps via the Work Guide, users require `Run Flows` permission.

---

## 5. Background Steps

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_concepts_steps.htm&type=5

- Calls an active **autolaunched flow** synchronously or asynchronously.
- No user interaction.
- Entry conditions are configurable (same options as approval steps).
- Use for: record updates, triggering notifications, record locking/unlocking, post-approval actions.

> Always use background steps (not approval steps) for DML operations or record updates. Approval steps should only present information for a decision.

---

## 6. Build a Flow Approval Process — Step-by-Step

**Sources:**
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_create_flow_approval_process_from_scratch.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_build_create_a_flow_approval_process.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_build_create_step_to_interact_with_reviewers.htm&type=5

### Option A: Approvals App (Autolaunched Only)

1. App Launcher → **Approvals**.
2. Click **Create Flow Approval Process** → **Start from Scratch**. Flow Builder opens.
3. Add stages, steps, and decisions.

### Option B: Setup → Flows (Autolaunched or Record-Triggered)

1. Setup → Quick Find → **Flows** → **New Flow**.
2. Select:
   - `Autolaunched Flow Approval Process (No Trigger)` for manual submission.
   - `Record-Triggered Flow Approval Process` for automatic trigger on record create/update.

### Configure a Record-Triggered Start Element

1. **Select Object** — the object whose records trigger the process.
2. **Configure Trigger** — `A record is created` / `A record is updated` / `A record is created or updated`.
3. **Set Condition Requirements:**

| Option | Behavior |
| --- | --- |
| None | Always runs |
| All Conditions Are Met (AND) | All conditions must be true |
| Any Condition Is Met (OR) | At least one condition must be true |
| Custom Condition Logic Is Met | Enter logic string up to 1,000 chars using numbers, AND, OR, NOT, parentheses |
| Formula Evaluates to True | Build a formula using Flow Formula Builder |

### Add Stages and Steps

```
Flow Approval Process
├── [Stage 1]
│   ├── Background Step (optional — e.g., lock record)
│   ├── Approval Step 1  ←── calls screen flow with approvalDecision output
│   └── Approval Step 2  ←── concurrent or sequential
├── [Decision] (optional routing logic)
├── [Stage 2]
│   ├── Approval Step 3
│   └── Background Step  ←── e.g., update fields, send notifications
└── [End]
```

1. Click `+` → **Stage** → enter label, API name, description.
2. Inside stage → click **Add Step** → `Approval Step` or `Background Step`.
3. For each **Approval Step**:
   - Set entry condition (when to start).
   - Under **Select an Action to Run**, pick an active screen flow containing `approvalDecision` text output variable. Provide required input values.
   - Under **Select an Approver**, set assignment type and assignee.
   - (Optional) Customize notification email using text templates — provide API names for Subject and Body.
   - Under **Select Where to Complete the Action**, select the variable with the record ID to review. Optionally: `Lock the record` and `Allow approver to edit the locked record`.
   - Set exit condition (when to complete the step).
4. **Go To connectors** — to connect to non-sequential elements: after an element, click `+` → **Connect to element** → click `+` on the target element.
5. **(Optional) Recall Path** — on the Start element click **Add Recall Path**. One stage with only background steps.
6. **(Optional) Fault Paths** — on a stage card click the actions icon → **Add Fault Path**. Add stage/decision elements. By default ends with End element; use Go To connectors to reconnect to the main path if needed.
7. Save.

### Add the Orchestration Work Guide Component

After building:
1. Open the Lightning record page for the object being reviewed in **Lightning App Builder**.
2. Add the **Orchestration Work Guide** component to the page layout.
3. Save and activate the page.

---

## 7. Deploy & Activate

**Sources:**
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_distribute_activate.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_distribute.htm&type=5

### Org-Wide Email Setup (Required Before Activation)

Setup → **Organization-Wide Addresses** → create an address with purpose **Automated Process Emails** or **Default No-Reply Address**. Reference it in **Process Automation Settings** → *Automated Process User Email Address*.

### Activate / Deactivate

- Only **one version** can be active at a time. Activating a new version deactivates the previous one.
- In-progress submissions continue on their original version.
- **From Flow Builder:** Open the flow approval process → click **Activate** or **Deactivate** on the button bar.
- **From Setup:** Flows list → quick menu → **View Details and Versions** → click **Activate** or **Deactivate** on the version row.

**Permissions required:**
- `Manage Flow` for autolaunched flow approval processes.
- `Manage Flow` + `View All Data` for record-triggered flow approval processes.

---

## 8. Make It Available to Users

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_deploy_allow_users_to_submit_approval.htm&type=5

**Record-triggered** processes run automatically — no additional user-facing setup needed beyond the Work Guide component.

**Autolaunched** processes require one of these:

### Option A: Request Approval Component (Internal or Experience Cloud)

- In **Lightning App Builder**, add the **Request Approvals** component to the record page.
- Configure so submitters can select an approver and add submission comments.
- For **Experience Cloud** sites: add it to the object details page in **Experience Builder**.

### Option B: Custom Button (Autolaunched)

1. **Object Manager** → target object → **Buttons, Links, and Actions** → **New Action**.
   - Action Type: **Flow**
   - Flow: select your active autolaunched flow approval process
   - Add to the object's page layout.

2. Or, use the **dynamic `Request an Approval` action** inside an autolaunched flow, then call that flow from a button. This supports the `Request an Approval` action element.

---

## 9. Resources Reference

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_ref_resources.htm&type=5

| Resource | Creatable | Description |
| --- | --- | --- |
| **Constant** | Yes | Fixed value usable throughout the process |
| **Decision Outcome** | No (auto-created with Decision element) | Boolean — `True` if that outcome path was executed |
| **Element** | No | Any Stage or Decision element; usable with `Was Visited` or `has error` operators in decision outcome criteria |
| **Formula** | Yes | Calculated value evaluated at run time |
| **Global Constants** | No (system-provided) | `EmptyString`, `True`, `False` |
| **Global Variable** | No (system-provided) | Org/user context: user ID, API session ID, `$Record`, etc. |
| **Step Resource** | No (auto-created) | Represents a step's status and output; used in conditions |
| **Text Template** | Yes | Reusable formatted text (HTML tags supported); used for custom email subjects/bodies |
| **Variable** | Yes | Mutable value; used to pass data between steps and as assignee references |

**Automatic Output:** Flow approval processes can query the status of any stage or step, and use output parameters from any step's referenced flow. Referenced outputs containing records or record collections are refreshed with latest values each time the orchestration run resumes.

---

## 10. Limits

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_considerations_limit.htm&type=5

| Limit | Enterprise / Unlimited / Performance / Developer |
| --- | --- |
| Versions per flow approval process | 50 |
| Active flow approval processes | 2,000 |
| Total flow approval processes (all versions) | 4,000 |
| Requirements per step entry/exit condition | 10 (use an evaluation flow for more) |
| Combined input values to a called flow | 32,768 characters (pass record ID instead of full record to avoid this) |
| Stages in a recall path | 1 (background steps only) |
| Step types in a recall path stage | Background steps only |

---

## 11. Considerations & Pitfalls

**Sources:**
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_considerations_orchestrations.htm&type=5
- https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_considerations.htm&type=5

### Entry/Exit Condition Requirements

- Resource and Value fields can contain flow approval process resources or global variables.
- To reference a record, you **must** select a field on the record — not the record itself.
- The referenced record must use fields from its **own** object, not fields from a related record.

### Requirement Resources That Do NOT Trigger Condition Evaluation

The following won't cause an orchestration run to re-evaluate when they change:
- Resource is a null record
- Resource is a record with an invalid ID
- Resource is a global variable in an autolaunched flow approval process
- Resource is a global variable other than `$Record` in a record-triggered flow approval process
- Resource is a record of an object that doesn't support change events

### Input Values Limit

If the combined inputs to a called flow exceed **32,768 characters** (can happen when passing full record objects), the process fails. **Fix:** pass only the record ID and use a `Get Records` element inside the called flow — this also ensures you always have the latest record data.

### No-Approval-Step Edge Case

If a flow approval process has **no approval steps**, every submission is **automatically approved**. Always include at least one approval step.

### Concurrent Approvals & Record Locking

Never apply the step-level `Lock the record` setting to concurrently running approval steps — it will throw an error. Manage locking via background steps (see [Section 3.5](#35-record-locking)).

### Evaluation Flows

- An evaluation flow is an autolaunched flow with process type `Evaluation Flow`.
- Must contain a predefined Boolean output variable `isOrchestrationConditionMet`.
- Used when you need more than 10 requirements, or complex condition logic.
- Must be created, tested, and activated before referencing in a flow approval process.

### Email Notifications on Dual Failure

When a flow called by a step fails and causes the orchestration run to fail, **two** error emails are sent: a flow error notification and a flow approval process error notification.

### Security

- Flow approval processes run in the **Automated Process User system context** by default.
- Flows called by approval steps run in the **context of the completing user**.
- Don't use `$User` as an approval step Resource assignee — it resolves to the system user in system context.

### `$Record_Prior` Not Refreshed

In record-triggered flow approval processes, `$Record_Prior` is **not** refreshed when the orchestration run resumes.

---

## 12. Troubleshooting & Debugging

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_troubleshoot.htm&type=5

### Debug Without Data Changes (Rollback Mode)

- Open the flow approval process in Flow Builder → Debug.
- Use **rollback mode** to simulate a run without committing DML.

### Debug With Data Changes

- Set a **start point** to skip earlier stages, an **end point** to stop before later stages, or both to isolate a middle segment.
- Set **mock outputs** for steps — a step with mock output does not run its referenced flow or make DB changes.
- You are the assigned user for all approval steps without mock output that meet their entry conditions.

### Error Emails

When an approval submission's orchestration run fails, Salesforce sends an error email to either:
- The **admin who last modified** the associated flow approval process, or
- The **Apex exception email recipients**.

Use the fault email to identify the failing orchestration run and diagnose the issue.

---

## 13. Managing Submissions & Work Items

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_manage.htm&type=5

### View Approval Work Items

- **Approval Work Items list view** — see all items; use quick menu to Approve/Reject without running the screen flow.
- **Orchestration Work Guide component** on the record page — shows all work items assigned to the current user for that record; sortable by last-modified date.

### View Approval Submissions

- **Approval Submissions list view** — see status, submission date, submitter, associated record.

### Reassign a Work Item

- Reassign from the Approval Submissions list view or the Work Items list view.
- Can only reassign items that are **assigned but not completed** on an **in-progress** submission.
- When reassigned, the new assignee and their delegates receive the **default** email notification (even if a custom notification is configured).

### Recall an In-Progress Submission

- From **Approval Submissions list view** → recall.
- Only **in-progress** submissions can be recalled.
- Who can recall: any user with `Approval Admin` system permission, or the original submitter.
- If a recall path is configured, it runs. The in-progress stage and its incomplete steps are canceled; open work items are also canceled. The recall path stage then runs. When complete, the submission is marked Recalled.

### Cancel an In-Progress Submission

- From **Approval Submissions list view** → cancel.
- Only **in-progress** submissions can be canceled.

### Email Notification Controls

- **Global toggle:** Process Automation Settings → `Send Approval Work Item Assignment Emails to Approvers`.
- **Per-flow-approval-process toggle:** Advanced setting `Don't send approver notifications` on the flow approval process.
- **Email approval response:** Process Automation Settings → `Enable email approval response` — lets approvers approve/reject by replying to their email.

### View Dependencies

In the Approvals app → Orchestration Version page → **Usage tab** — shows flows called by the flow approval process and flows that call it via an Action element.

---

## 14. Migration from Legacy Approval Processes

**Source:** https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals_considerations_migrating_from_legacy_approval_processes.htm&type=5

| Feature | Legacy Approval Processes | Flow Approval Processes |
| --- | --- | --- |
| Submit for Approval Button | Auto-added via wizard | No button for record-triggered; custom button or Request Approval component needed for autolaunched |
| Entry Conditions | Set on approval process; shows error if unmet | Record-triggered: entry criteria on Start element. Autolaunched: use background steps + Decision in first stage |
| Manually Select Next Approver | Supported | **Not supported** — all assignees defined in the process; use a Resource variable resolved at run time |
| Recall | Submitter can recall if enabled | Submitter or `Approval Admin` can recall |
| Approver Delegates | Configurable | Supported — delegates always receive notifications and can complete work items |
| Recall Actions | Defined in Setup UI | Add a Recall path to the Start element (1 stage, background steps only) |
| Approval Email Template | Required, defined per process | Optional — uses default or text templates per step |
| Emails to Group/Queue Members | Each member is a direct recipient | Depends on `Enable email approval response` setting: To (enabled) or BCC (disabled) |

---

## 15. Metadata Deployment Reference

### Flow Approval Process Metadata Type

Flow Approval Processes deploy as **Flow** metadata (`.flow-meta.xml`) with a `processType` of `Orchestrator` (record-triggered) or `ApprovalOrchestration` (autolaunched).

**File path pattern:**
```
force-app/main/default/flows/<FlowApiName>.flow-meta.xml
```

**Key `processType` values:**
| Value | Type |
| --- | --- |
| `Orchestrator` | Record-Triggered Flow Approval Process |
| `ApprovalOrchestration` | Autolaunched Flow Approval Process |
| `Flow` | Screen Flow (called by approval steps) |
| `AutoLaunchedFlow` | Autolaunched Flow (called by background steps) |
| `EvaluationFlow` | Evaluation Flow (used in entry/exit conditions) |

**Key metadata fields on the Flow type relevant to approval processes:**

| Field | Purpose |
| --- | --- |
| `processType` | Identifies the flow type (see above) |
| `status` | `Active` or `Draft` — set to `Active` to deploy as active |
| `start` | Start element; for record-triggered, contains `object`, `triggerType`, and `filterConditions` |
| `orchestratedStages` | Array of stage elements |
| `steps` | Within each stage: approval steps and background steps |
| `actionCalls` | Background step flow invocations or action calls (e.g., Lock Record) |
| `decisions` | Decision elements with outcome criteria |
| `textTemplates` | Text template resources (for custom email subjects/bodies) |
| `variables` | Variable resources (including assignee references) |
| `formulas` | Formula resources |
| `constants` | Constant resources |

> **Developer API links (JS-rendered — open in browser):**
> - Flow Metadata: https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_visual_workflow.htm
> - Orchestration Metadata: https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_orchestration.htm

### Deploying via SF CLI

```bash
# Retrieve a specific flow approval process
sf project retrieve start --metadata "Flow:<FlowApiName>"

# Deploy a flow approval process
sf project deploy start --metadata "Flow:<FlowApiName>"

# Deploy and activate in one step (set status: Active in the .flow-meta.xml)
sf project deploy start --metadata "Flow:<FlowApiName>"
```

### Required Companion Metadata

When deploying a flow approval process, also deploy or verify:
- **Screen flows** called by approval steps (`processType: Flow`)
- **Autolaunched flows** called by background steps (`processType: AutoLaunchedFlow`)
- **Evaluation flows** used in conditions (`processType: EvaluationFlow`)
- **Lightning page** with the `Orchestration Work Guide` component (`FlexiPage`)
- **Custom buttons/actions** for autolaunched process submission (`QuickAction` or `WebLink`)
- **Org-wide email address** (configured manually — not deployable via Metadata API)
- **Process Automation Settings** (`Enable email approval response`, `Send Approval Work Item Assignment Emails`) — configure in target org after deployment
