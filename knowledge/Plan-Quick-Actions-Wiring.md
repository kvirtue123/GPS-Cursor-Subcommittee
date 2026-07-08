---
feature_area: agentforce-service-assistant
status: folded-into-catalog
catalog_id: P4
catalog_ref: patterns/templates/phased-delivery-structure.md
note: "Its '## Deploy Commands' and '## What Is NOT Changing' sections were already folded into P4 (phased-delivery-structure.md) with provenance. Do not re-catalog as a new asset."
---

# Plan: Full Quick Actions Wiring for Service Assistant

## Context

The current build has 7 subagent XMLs (`GenAiPlugin`, `pluginType: Topic`) whose instructions say **"You must call HECM_Lookup_Compliance_Status..."** — treating the 3 Agentforce action plugins as autonomous agent calls. Per Salesforce Service Assistant docs, this is architecturally wrong: **Service Assistant never executes Agentforce actions on its own. All actions are treated as grounding text (instructions), not autonomous invocations.** The only mechanism that surfaces as a clickable button inside a service plan step is a **Case Quick Action**.

This plan wires the 3 existing workflows as Case Quick Actions so they appear as buttons in service plan steps, then rewrites the 7 subagent instructions to align with the Service Assistant grounding model.

---

## Scope of Changes

### Phase 1 — New Case Quick Action metadata (3 new files)

Each quick action wraps an existing workflow. All must have a single `CaseId` / `recordId` input.

| Quick Action API Name | Type | Wraps | Notes |
|---|---|---|---|
| `Case.HECM_Run_Compliance_Lookup` | Flow | New Screen Flow `HECM_Compliance_Status_Lookup_Screen` | Apex invocable returns data — needs a screen to display it to the rep |
| `Case.HECM_Create_Compliance_Followup` | Flow | `FHA_Create_Compliance_Follow_Up_Task` (existing auto-launched Flow) | Existing Flow likely works as-is; verify `recordId` input variable mapping |
| `Case.HECM_Privacy_SOP_Checklist` | Flow | New Screen Flow `HECM_Privacy_SOP_Checklist_Screen` | Prompt template calls require LWC or a Flow with a GenAI step |

**New files to create:**
- `force-app/main/default/quickActions/Case.HECM_Run_Compliance_Lookup.quickAction-meta.xml`
- `force-app/main/default/quickActions/Case.HECM_Create_Compliance_Followup.quickAction-meta.xml`
- `force-app/main/default/quickActions/Case.HECM_Privacy_SOP_Checklist.quickAction-meta.xml`

### Phase 2 — New Screen Flow for compliance lookup (1 new file)

`HECM_Compliance_Status_Lookup_Screen` Flow:
- Input: `recordId` (String, CaseId)
- Action: Call `HECMLookupComplianceStatus` Apex invocable
- Screen: Display all 6 compliance statuses + summary string to the rep
- Save as a Screen Flow and reference it from the quick action

**New file:**
- `force-app/main/default/flows/HECM_Compliance_Status_Lookup_Screen.flow-meta.xml`

### Phase 3 — Case Page Layout (1 modified file)

Add all 3 quick actions to the `Salesforce Mobile and Lightning Experience Actions` section of the Case layout. This is required for them to appear in the Quick Actions Manager and in drafted service plans.

**File to modify:**
- `force-app/main/default/layouts/Case-Case Layout.layout-meta.xml` (check if this file exists; create/modify accordingly)

### Phase 4 — Rewrite 7 subagent instruction XMLs

Rewrite the specific instructions that reference Agentforce actions. Use the **Dependent Instructions approach** (Approach 2) since the HUD use cases are tightly coupled to specific workflows.

**Instruction rewrites by subagent:**

| Subagent | Instruction | Current (wrong) | Rewritten (correct) |
|---|---|---|---|
| All 5 compliance subagents | Instruction 01 | "You must call HECM_Lookup_Compliance_Status with Case.FHA_Case_Number__c..." | "Use the **Run HECM Compliance Lookup** action to retrieve the borrower's current compliance statuses before providing any guidance. The result shows all 6 annual compliance statuses and a plain-language summary." |
| Annual Occupancy Certification | Instruction 07 | "You must call HECM_Create_Compliance_Followup_Task with the active Case Id..." | "When occupancy certification status is Outstanding or Overdue, include a step for the rep to use the **Create Compliance Follow-Up Task** action. This schedules a 30-day follow-up task automatically. Do not skip this step when status is Outstanding or Overdue." |
| Property Insurance Compliance | Instruction 04 | "You must call HECM_Run_Privacy_SOP_Checklist..." | "For calls involving the hazard insurance declaration page, include a step for the rep to use the **HECM Privacy SOP Checklist** action. It walks through the 6-item HERMIT checklist and generates the compliant SR description." |

**Files to modify (all 7):**
- `force-app/main/default/genAiPlugins/HECM_Annual_Occupancy_Certification.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_Eligible_NBS_Annual_Certification.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_HOA_Ground_Rent_Compliance.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_Non_Borrowing_Spouse_Identification.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_Property_Insurance_Compliance.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_Property_Tax_Compliance.genAiPlugin-meta.xml`
- `force-app/main/default/genAiPlugins/HECM_Verify_Caller_Identity.genAiPlugin-meta.xml` (verify — may not need changes if it has no action calls)

**Also fix while in these files:**
- `HECM_Property_Tax_Compliance` Instruction 05: replace "address in the knowledge article" with the explicit Compu-Link submission address used in the other plugins

### Phase 5 — Manual: Quick Actions Manager (UI steps, not metadata)

After deploying, complete in the legacy Agentforce Builder:

1. Go to Setup → Agentforce Agents → Service Assistant → Open in Builder → **Quick Action** (left rail)
2. Deactivate the agent
3. Turn the Quick Actions feature on
4. For each of the 3 quick actions, click **Manage** and add instructions:

   **Run HECM Compliance Lookup:**
   > Retrieves the current annual compliance status for all six HECM requirements from HERMIT for the active case. Use this action at the start of any HECM annual compliance call to confirm the borrower's status before providing guidance. Once executed, the compliance dashboard shows all six statuses and a plain-language summary.

   **Create Compliance Follow-Up Task:**
   > Schedules a 30-day follow-up task on the active case when the borrower's annual occupancy certification status is Outstanding or Overdue. Use this action after confirming an outstanding or overdue occupancy certification. Once executed, a task is created and assigned to the HECM Servicing Compliance Follow-Up queue with a due date 30 days from today.

   **HECM Privacy SOP Checklist:**
   > Runs the HERMIT Privacy SOP walkthrough for calls involving hazard insurance declaration page requests. Use this action when the borrower or representative is requesting access to or discussing the insurance declaration page. Once executed, the checklist walks the rep through 6 HERMIT verification steps and generates a compliant SR description.

5. Assign each quick action to the relevant subagents
6. Reactivate the agent

---

## Deploy Commands

```bash
# Phase 1–3: Quick actions + layout
sf project deploy start \
  --metadata "QuickAction:Case.HECM_Run_Compliance_Lookup" \
  --metadata "QuickAction:Case.HECM_Create_Compliance_Followup" \
  --metadata "QuickAction:Case.HECM_Privacy_SOP_Checklist" \
  --metadata "Flow:HECM_Compliance_Status_Lookup_Screen" \
  --metadata "Layout:Case-Case Layout" \
  --target-org FHA-Demo --wait 15

# Phase 4: Subagent instruction rewrites
sf project deploy start \
  --metadata "GenAiPlugin:HECM_Annual_Occupancy_Certification" \
  --metadata "GenAiPlugin:HECM_Eligible_NBS_Annual_Certification" \
  --metadata "GenAiPlugin:HECM_HOA_Ground_Rent_Compliance" \
  --metadata "GenAiPlugin:HECM_Non_Borrowing_Spouse_Identification" \
  --metadata "GenAiPlugin:HECM_Property_Insurance_Compliance" \
  --metadata "GenAiPlugin:HECM_Property_Tax_Compliance" \
  --metadata "GenAiPlugin:HECM_Verify_Caller_Identity" \
  --target-org FHA-Demo --wait 15
```

---

## Verification

1. Deploy outputs show `State: Created` for all 3 quick actions and the new Flow
2. Quick actions appear in Setup → Object Manager → Case → Buttons, Links, and Actions
3. Quick actions appear in the Case page layout (Mobile and Lightning Experience Actions section)
4. Quick actions appear in the Quick Actions Manager in the legacy Agentforce Builder
5. After adding instructions and assigning to subagents in the Quick Actions Manager, draft 2–3 service plans for Annual Compliance case type
6. Confirm quick action buttons appear inside the appropriate plan steps (Run Compliance Lookup at step 1, Create Follow-Up Task when occupancy status is outstanding)
7. Click each button and verify the underlying workflow fires correctly

---

## What Is NOT Changing

- The 3 existing Agentforce Action plugins (`HECM_Lookup_Compliance_Status`, `HECM_Create_Compliance_Followup_Task`, `HECM_Run_Privacy_SOP_Checklist`) remain in the codebase. Per SF docs, they will be treated as additional grounding instructions — no harm, no foul.
- The `HECM_Verify_Caller_Identity` subagent has no action calls — its instructions are well-structured and require only review, not rewrite.
- The `HECM_Draft_Service_Plan_Email` prompt template is unaffected.
- The Apex class `HECMLookupComplianceStatus` is unchanged — the new Screen Flow calls it as-is.
