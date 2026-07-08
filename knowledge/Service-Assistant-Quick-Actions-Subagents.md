---
feature_area: agentforce-service-assistant
status: duplicate-of-catalog
catalog_id: P22
catalog_ref: patterns/canonical/service-assistant-quick-actions.md
note: "Already cataloged as P22 (Service Assistant quick-actions/subagents grounding model). Kept here for local browsing — do not re-catalog."
---

# Agentforce Service Assistant: Quick Actions & Subagents Reference

**Sourced from Salesforce Help, April 2026**

---

## Critical Architecture Insight

> **Agentforce actions are NOT required and NOT used the same way in Service Assistant.**
>
> Per SF docs: *"Agentforce actions aren't required in Service Assistant. Actions are specific tasks that AI agents execute for customers. Service Assistant doesn't complete any direct actions for the customer or service rep. Instead, all steps in a service plan and any actions required for those steps are carried out directly by the service rep. If you add any Agentforce actions, they're treated like instructions."*

This means:

| Mechanism | Role in Service Assistant |
|---|---|
| **Subagents (formerly Topics) + Instructions** | PRIMARY grounding — determines plan steps |
| **Quick Actions** | OPTIONAL grounding — surfaces as buttons inside plan steps |
| **Agentforce Actions** | Treated like additional instructions, not executed autonomously |
| **Knowledge (Data Libraries)** | OPTIONAL — fills in knowledge article content as plan steps |

---

## What Quick Actions Are (and Are Not)

Quick actions in Service Assistant:
- Appear as **clickable buttons inside service plan steps** so reps don't have to leave the component
- Are **grounding sources** — they influence what steps get generated in a plan
- Are assigned to **subagents (topics)** via the Quick Actions Manager in the (legacy) Agentforce Builder
- Require **instructions** that tell Service Assistant *when* to surface them in a plan step
- Must be added to the **Case page layout** (Salesforce Mobile and Lightning Experience Actions section) to appear

Quick actions are NOT:
- Autonomous agent actions (the rep clicks them, not the agent)
- Self-executing
- Available in the new Agentforce Builder — **legacy Agentforce Builder only**

---

## Supported Quick Action Types

### Standard Case Object Quick Actions

| Quick Action | Notes |
|---|---|
| Case Comment | Standard |
| Close Case | Standard — provided instructions below |
| Change Owner | Standard |
| Change Status | Standard |
| Log a Call | Standard |
| Send Email | Standard — provided instructions below |
| Summarize Case | Requires Work Summaries User permission set + Einstein Work Summaries for Case (beta) enabled |

### Custom Case Object Quick Actions

| Type | Notes |
|---|---|
| Create a Record | Supported |
| Lightning Web Component | Screen actions only |
| Log a Call | Supported |
| Flow | Requires Run Flows app permission for admin, reps, and ServicePlanner User |
| Send Email | Supported |
| Update a Record | Supported |
| Visualforce | Requires Visualforce page access/view permissions |

**Constraint:** All quick actions used in Service Assistant must have only a single `CaseId` input.

---

## Salesforce-Provided Quick Action Instructions

Use these as starting points and modify to fit the HUD/FHA context.

### Send Email
> Drafts and sends an email to a specified recipient directly from a service plan step. The email can be used for customer communications, including, but not limited to, providing status updates, conveying resolution details, and requesting or supplying additional information during case management or follow-up.

### Close Case
> Closes the case when the required steps to resolve issues are completed and no further action is needed. When executed, the support process is formally concluded and no additional steps can be performed.

### Summarize Case
> Creates a case summary that incorporates Service Plan information, such as completed steps and the final outcome. Intended for use when the case is nearing resolution, the case is resolved, or the customer or service rep requests a summary. Service reps initiate the quick action directly from a service plan step for an in-workflow experience.

---

## How to Write Effective Quick Action Instructions

### Two Formats: Condensed vs. Detailed

**Condensed (single conditional sentence):**
```
Launches a flow to initiate and submit a billing dispute when a customer reports
an incorrect or unrecognized charge on their account.
```

**Detailed (three-part structure):**
```
Launches a flow to document and submit a customer's billing dispute. This action
should be used when a customer reports an incorrect or unrecognized charge on
their account. Once executed, the flow creates a dispute record, places a
temporary hold on the specified charge, and notifies the finance department for review.
```

### Key Principles

1. **Specific name** — state the exact use case (e.g., "HECM Insurance Status Notification" not "Send Email")
2. **Define the general action** — what does the action do at a high level?
3. **State conditions** — when should this action surface in a plan step?
4. **State requirements/outcomes** — what happens once executed?

### Less Effective vs. More Effective

| Less Effective | More Effective |
|---|---|
| Name: Update Order Status | Name: Mark Order As Shipped |
| Generic, no specific business context | Tied to specific workflow and conditions |
| May still surface but inconsistently | Surfaces reliably in the right plan steps |

---

## Combining Quick Actions with Subagent Instructions

There are two approaches. **Try both** — no single method guarantees surfacing.

### Approach 1: Independent Instructions (Recommended)

- **Subagent instruction**: describes high-level policy/process, does NOT mention the quick action name
- **Quick action instruction**: self-contained, defines function + conditions + outcome
- The two never reference each other — Service Assistant semantically matches them

**Example:**

| | Content |
|---|---|
| **Subagent (Billing Issue) Instruction** | When a customer has questions about a specific charge on their bill or indicated the charge is incorrect, initiate a billing dispute and verify the transaction details in the payment system. |
| **Quick Action (Initiate Billing Dispute)** | Launches a flow to initiate and submit a billing dispute when a customer reports an incorrect or unrecognized charge on their account. |

### Approach 2: Dependent Instructions

- **Subagent instruction**: explicitly names the quick action ("Use the Notification of Refund tool to...")
- **Quick action instruction**: detailed, mirrors the conditions in the subagent instruction
- Works via tight coupling; reduces ambiguity but requires careful maintenance

**Example:**

| | Content |
|---|---|
| **Subagent (Process Refund) Instruction** | When the refund process is completed, use the Notification of Refund tool to inform the customer that the refund has been processed and provide any additional information they may need. |
| **Quick Action (Notification of Refund)** | Concludes the refund process by drafting and sending a notification email to the customer. This action is used directly from a service plan step once refund is initiated. It provides the customer with final confirmation and transaction details, and a note is logged to the case record. |

---

## Subagents (Topics) — Role and Guidelines

> **Note:** In April 2026, Salesforce renamed "topics" to "subagents." Functionality is unchanged. You may see both terms in the UI during transition.

### What Subagents Do in Service Assistant

Subagents classify incoming cases and determine which instructions (and therefore which plan steps) are relevant. They are the **primary required grounding source**.

Unlike autonomous Agentforce agents, Service Assistant subagents don't execute tasks — they provide the structured guidance that the service rep follows.

### Guidelines for Writing Subagents

| Guideline | Detail | Example |
|---|---|---|
| Clear and specific scope | Function as a label for the exact case type | Use "Product Inquiry" not "Customer Inquiry" |
| No overlap between subagents | Overlapping subagents make classification unreliable | Consolidate "Manage Reservations" + "Confirm Reservation" into "Reservation Management" |
| Keep subagents singular | One concept per subagent | Separate "Returns and Exchanges" into "Returns" + "Exchanges" |
| Encompassing descriptions | List variations/keywords in the description field | See HECM example below |

### Subagent Description — Include Synonyms and Subtypes

```
Subagent: HECM Insurance Status
Description: Guide service reps in helping borrowers resolve questions about
their HECM loan insurance declaration status. Questions are related to FHA
insurance endorsement, MIP (Mortgage Insurance Premium) status, Case Number
assignment, FHA case status inquiry, and insurance certificate requests.
```

### Guidelines for Writing Subagent Instructions

| Guideline | Technique |
|---|---|
| Use conditional statements | "If the borrower provides X, then..." |
| State actions + expected outcomes | "After verifying, confirm the status in the system..." |
| Specify your policies explicitly | Reference your actual SOP, not generic guidance |
| Mark required steps | Use "Always include this as a step in the plan" |
| One piece of information per instruction | Don't combine multiple policy points in one instruction |
| Omit PII and sensitive data | Don't include names, IDs, or confidential specifics |

### Instruction Language Patterns

**For mandatory steps:**
- "Always", "Must", "Required", "Confirm", "Verify", "At all times"

**For sequential steps:**
- "After you have...", "After completing...", "After verifying..."

**For conditional steps:**
- "If..., then...", "When..., then...", "Provided that..., then..."

---

## Sample Subagent: Return Request (Salesforce Reference)

```
Label: Return Request

Description: Assist service reps with handling customer questions related to
return requests. Returns requests can also be called return processing, returns,
refunds, return inquiries, and authorize return. Questions are related to return
transactions, which include proof of return, refund status, return documentation,
return policy, return window, and return resolution steps.

Scope: Your job is to assist service reps in resolving customer inquiries
regarding returns processing. Ensure they adhere to all internal policies and
procedures. You must not handle inquiries unrelated to the return of goods.

Instruction 1: Make sure the customer has provided the required information:
valid return date, customer name, and receipt number or order number.
This step must always be in a plan for cases dealing with returns.

Instruction 2: After you have the required information, you must confirm the
purchase in the return management system.

Instruction 3:
- If the customer provides the full Return Number, search the return in the
  Returns Management System (RMS) using the return number and return date.
- If the customer provides the order number, search the return in the Order
  Management System (OMS) using the order number and return date.
- If the customer provides only the customer name, search the return in
  Salesforce using the customer name and return date.

Instruction 4: [handle not-found and escalation scenarios...]

Instruction 5: [confirm return condition requirements...]

Instruction 6: After gathering all information, send a response using the
Return Request Template. This step must always be in a plan.

Instruction 7: If the customer isn't happy with the outcome, offer to escalate
to a higher-level support team. Document the customer's concerns and steps taken.
```

---

## Setup Summary

### Prerequisites
1. Service Assistant is **legacy Agentforce Builder only** — not the new builder
2. Quick actions must be added to the **Case Page Layout** (Mobile and Lightning Experience Actions)
3. Custom quick actions must have a single `CaseId` input
4. You cannot assign quick actions to **standard Agentforce topics** — you must create versions of them
5. Deactivate the agent before assigning quick actions in the Quick Actions Manager

### Quick Actions Manager Location
Setup → Agentforce Agents → Service Assistant → Open in Builder → **Quick Action** (left rail)

### Required Permissions

| Role | Permissions Needed |
|---|---|
| Admin (setup) | Customize Application or System Admin + Agentforce Default Admin + Service Planner Builder |
| Summarize Case (admin) | + Work Summaries User permission set |
| Service reps (use) | Service Planner User permission set |
| Summarize Case (reps) | + Work Summaries User permission set |
| Flow quick actions | Run Flows app permission |
| Visualforce quick actions | Visualforce page access/view permissions |

---

## Implications for HUD FHA Demo Rewrite

### What This Means for Instructions

Because Agentforce **actions** in Service Assistant are treated as additional instructions (not executed autonomously), the instruction-writing approach must shift:

1. **Subagent instructions** = the primary grounding; these should describe borrower case scenarios (HECM status, MIP inquiries, FHA endorsement questions) with conditional, policy-specific steps
2. **Quick actions** = optional buttons that surface in plan steps; each needs its own instruction describing when to use it
3. **Actions listed in the agent** = treated as instructions, not automated execution — do not expect them to fire autonomously

### Quick Action Naming for HUD FHA Context (Suggested)

Instead of generic names, use specific names tied to HECM workflows:

| Generic Name | Suggested Specific Name |
|---|---|
| Send Email | Notify Borrower of Insurance Status |
| Update a Record | Update FHA Case Status |
| Log a Call | Log Borrower Inquiry |
| Close Case | Close Resolved Insurance Inquiry |

### Key Constraint for Grounding (from memory)
Grounding fields require **String or Text Area** field types — not picklists. This applies to Service AI Grounding field selection as well.

---

## Source URLs

- [Agentforce Service Assistant Overview](https://help.salesforce.com/s/articleView?id=service.sp_intro.htm&type=5)
- [Ground Service Assistant](https://help.salesforce.com/s/articleView?id=service.sp_comps.htm&language=en_US&type=5)
- [Grounding with Subagents](https://help.salesforce.com/s/articleView?id=service.sp_topics_start.htm&language=en_US)
- [Grounding with Quick Actions](https://help.salesforce.com/s/articleView?id=service.sp_qa.htm&language=en_US)
- [Set Up Quick Actions](https://help.salesforce.com/s/articleView?id=service.sp_qa_setup.htm&type=5)
- [Guidelines for Creating Quick Actions](https://help.salesforce.com/s/articleView?id=service.sp_qa_gbp.htm&type=5)
- [Quick Actions with Topics](https://help.salesforce.com/s/articleView?id=service.sp_qa_topics.htm&type=5)
- [Quick Actions Sample Library](https://help.salesforce.com/s/articleView?id=service.sp_qa_sample_library.htm&type=5)
- [Draft Service Plan Emails](https://help.salesforce.com/s/articleView?id=service.sp_email_create.htm&type=5)
