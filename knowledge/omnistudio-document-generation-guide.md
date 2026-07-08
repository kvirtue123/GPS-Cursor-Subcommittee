---
feature_area: omnistudio
status: duplicate-of-catalog
catalog_id: P15
note: "Already cataloged as P15 (OmniStudio Document Generation recipe). Kept here for local browsing — do not re-catalog."
---

# Omnistudio Document Generation — Implementation Guide

This guide consolidates the official Salesforce documentation needed to build a custom **Generate Document** feature using the Omnistudio Document Generation framework. It covers the custom button setup, the `fndSingleDocxLwc` Omniscript and its step properties, customization to preselect templates, the `fndmultiPDFConvertLwc` Omniscript for converting client-side documents to PDF, and the browser-based previewer configuration.

> Source documents (Salesforce Help):
> - [Create a Custom Generate Document Button](https://help.salesforce.com/s/articleView?id=ind.doc_gen_create_a_custom_generate_document_button_394436.htm&type=5)
> - [Customize the fndSingleDocxLwc Omniscript](https://help.salesforce.com/s/articleView?id=ind.doc_gen_customize_fndsingledocxlwc_omniscript_394579.htm&type=5)
> - [Step Properties for the fndSingleDocxLwc Omniscript](https://help.salesforce.com/s/articleView?id=ind.doc_gen_omniscript_step_properties_394328.htm&type=5)
> - [Convert Client-Side Documents into PDF Files (fndmultiPDFConvertLwc)](https://help.salesforce.com/s/articleView?id=ind.doc_gen_foundation_docgenerationsample_fndmultipdfconvertlwc_omniscript__394874.htm&type=5)
> - [Configure Browser-Based Preview for Omnistudio Document Generation](https://help.salesforce.com/s/articleView?id=ind.doc_gen_permissions_browser_based_previewer_docgen_two.htm&type=5)

---

## 1. Create a Custom Generate Document Button

Generate documents from any object (contract, order, quote, work order, opportunity, etc.) by integrating the `fndSingleDocxLwc` Omniscript into a custom button. This Omniscript generates single client-side documents from Microsoft Word `.docx` or Microsoft PowerPoint `.pptx` templates.

### 1.1 Retrieve the Lightning URL from the fndSingleDocxLwc Omniscript

1. Open the **fndSingleDocxLwc** Omniscript.
2. From the **Actions** dropdown list, click **How to Launch**.
3. Scroll to **Lightning**, and copy the URL starting from `/lightning/cmp/omnistudio`.

Example URL:

```
/lightning/cmp/omnistudio__vlocityLWCOmniWrapper?c__target=c:docGenerationSampleFndSingleDocxLwcEnglish&c__layout=lightning&c__tabIcon=custom:custom18
```

4. Click **Done**.

### 1.2 Create the Custom Generate Document Button

1. In **Setup**, click **Object Manager**.
2. Click the object you want the button on (for example, **Work Order**).
3. Click **Buttons, Links, and Actions**.
4. Click **New Button or Link** and enter:

| Field | Value |
| --- | --- |
| **Label** | Generate Document |
| **Name** | Generate_Document |
| **Display Type** | Detail Page Button |
| **Behavior** | Display in new window |
| **Content Source** | URL |
| **Button or Link URL** | Paste the Lightning URL copied from the Omniscript |

5. Save and click **OK**.

### 1.3 Add URL Parameters

Append parameters to the Lightning URL to control the Omniscript:

- **Template Type** — append `&c__TemplateType=MicrosoftWord`:
  ```
  /lightning/cmp/omnistudio__vlocityLWCOmniWrapper?c__target=c:docGenerationSampleFndSingleDocxLwcEnglish&c__layout=lightning&c__tabIcon=custom:custom18&c__TemplateType=MicrosoftWord
  ```

- **Object ID** — append `&c__ObjectId=` and use **Insert Field** to insert the record ID merge field (for example `{!WorkOrder.Id}`):
  ```
  /lightning/cmp/omnistudio__vlocityLWCOmniWrapper?c__target=c:docGenerationSampleFndSingleDocxLwcEnglish&c__layout=lightning&c__tabIcon=custom:custom18&c__TemplateType=MicrosoftWord&c__ObjectId={!WorkOrder.Id}
  ```

> **Note:** During document generation the work-ID parameter is replaced with the actual record ID.

### 1.4 Add the Button to the Page Layout

1. **Setup → Object Manager → *yourObject* → Page Layouts → *yourLayout* → Buttons**.
2. Drag **Generate Document** to the **Custom Buttons** section.
3. Save. The button appears in the **Actions** menu of the record.

### 1.5 Generate the Document

1. On a record, click **Actions → Generate Document**. The fndSingleDocxLwc Omniscript launches with **Object ID** and **Template Type** prefilled.
2. Click **Next**.
3. Select a template and click **Next**. *(Optional: hide this step — see [Section 2](#2-customize-the-fndsingledocxlwc-omniscript).)*
4. Choose generation options and click **Next**. The document is generated. You can download `.pdf` and `.docx` versions, attach them to the record, or email them.

---

## 2. Customize the fndSingleDocxLwc Omniscript

Customize the Omniscript to preselect a default template and **hide the Pick a Template window** during generation. This Omniscript can then be invoked from a custom **Generate Document** button on any object.

### 2.1 Prerequisites

Complete steps 1–3 of [Section 1](#1-create-a-custom-generate-document-button).

### 2.2 Add the Template ID Parameter to the Custom Button

1. **Setup → Object Manager → *yourObject* → Buttons, Links, and Actions → Generate Document → Edit**.
2. Append `&c__templateId=<templateRecordId>` to the URL.

Example:

```
/lightning/cmp/omnistudio__vlocityLWCOmniWrapper?c__target=c:docGenerationSampleFndSingleDocxLwcEnglish&c__layout=lightning&c__tabIcon=custom:custom18&c__TemplateType=MicrosoftWord&c__ObjectId={!WorkOrder.Id}&c__templateId=2dt5f0000004C98AAE
```

3. Save.

### 2.3 Modify the Omniscript to Set Default Values

1. Clone fndSingleDocxLwc by clicking **New Version**.

2. **Hide the Pick a Template window** — In the **PickTemplate** step:
   - Go to **Properties → Conditional View** and click **Show Element if True**.
   - In **Edit Show Hide Rules**, enter:
     - **Field:** `templateId`
     - **Operator:** `Equal To`
     - **Value:** *(blank)*
   - Save.

3. **Preselect the template** — In the **SetValues** step, click **Edit Properties As JSON** and replace the `elementValueMap` node with:

```json
"elementValueMap": {
  "preSelectedTemplate": {
    "Id": "=%templateId%",
    "TemplateType": "=%TemplateType%"
  },
  "selectedTemplate": "=IF(%templateId% == null, %PickTemplate|1:SelectTemplate|1%, %preSelectedTemplate%)",
  "templateType": "=%TemplateType%"
},
```

4. Click **Close JSON Editor**.

5. **Pass the preselected template forward** — In the **SetGenerationOptions** step, click **Add New Element Value** and enter:
   - ✅ **Use Expression For Value**
   - **Element Name:** `selectedTemplate`
   - **Expression:** `IF(%templateId%!=NULL, %preSelectedTemplate%, %selectedTemplate%)`

6. Save, click **Activate Version**, then **Done**.

### 2.4 End-to-End Behavior

When the user clicks **Generate Document**:

1. Object ID and Template Type are populated automatically.
2. Click **Next** — the **Pick a Template** window is skipped because `templateId` is supplied via the URL.
3. Choose generation options → click **Next** → document is generated.

---

## 3. Step Properties for the fndSingleDocxLwc Omniscript

The Omniscript runs its steps sequentially. Salesforce recommends cloning or creating a new version before changing any step.

| Omniscript Step | Properties |
| --- | --- |
| **Script Configuration** | **Type:** `docGenerationSample` • **SubType:** `fndSingleDocxLwc` |
| **EnterObject** | **ObjectId** — Salesforce record ID (assigned to `contextID` in **SetValues**). **TemplateType** — `Microsoft Word .DOCX Template` or `Microsoft PowerPoint .PPTX Template`. |
| **GetDocumentTemplates** | Invokes the `DocGenSample-ExtractDocumentTemplatesLWC` Omnistudio Data Mapper to retrieve all active DOCX/PPTX templates. |
| **PickTemplate** | Displays the templates in the `clmSelectableItems` LWC. |
| **SetValues** | Sets `selectedTemplate` and `templateType` for the **Generate Document** step. |
| **GenerationOptions** | See properties below. |
| **Set Generation Options** | Maps the values from **GenerationOptions** to the variables passed to **Generate Document**: `attachFileFormat`, `docGenerationMechanism`, `documentViewer`, `downloadFileFormat`, `pdfGenerationSource`, `useTemplateDRExtract`, `documentGenerationFontSource`, `insertSpaceforRepeatingContentTokens`, `documentTitle`. |
| **Generate Document** | Uses the `clmOsDocxGenerateDocument` LWC to generate and display the document. **Email Word/PowerPoint** and **Email PDF** options send to the current user's email. The LWC invokes the `DocgenAppHandler` Apex class. |

### 3.1 GenerationOptions Step — Field Reference

| Field | Description |
| --- | --- |
| **DownloadFileFormat** | Document type for download: `All` (default), `Microsoft Word`, `Microsoft PowerPoint`, `PDF`, or `None`. |
| **AttachFileFormat** | Document type for attachment: `Microsoft Word` (default), `Microsoft PowerPoint`, `PDF`, `Word, PDF`, `PowerPoint, PDF`, or `None`. |
| **DocumentViewer** | Preview format: `PDF` or `None`. `None` disables preview. |
| **DocumentTitle** | Title for the generated document. **Mandatory** in Document Generation 2.0. |
| **UseTemplateDRExtract** | Whether to use the Data Mapper Extract associated with the selected template. |
| **DocGenerationMechanism** | Read-only, set to `ClientSide`. Customize the Omniscript to make selectable. |
| **PdfGenerationSource** | Read-only, set to `ClientSide`. Customize the Omniscript to make selectable. |
| **Insert Space for Repeating Content Tokens** | Whether the repeating content token's output is space-separated in generated documents. |
| **documentGenerationFontSource** | Source of font for rendering values. Values: `Rich Text Editor Font` (default — uses font in the rich-text field data) or `Document Font` (uses the template font). |
| **Keep Intermediate** *(optional)* | Whether intermediate DOCX/PPTX files generated for PDF conversion are retained. When `attachFileFormat` and `downloadFileFormat` are both `pdf`, setting `keep-intermediate=false` deletes the intermediate file after conversion to reduce storage. Supported by `singleDocxLwc`, `singleWebLwc`, `multiDocxLwc` Omniscripts via `ClmOsDocxGenerateDocument`, `ClmOsWebGenerateDocument`, and `ClmOsMultiDocxGenerateDocument`. Default: `true`. |

---

## 4. Convert Client-Side Documents into PDF (fndmultiPDFConvertLwc)

The **fndmultiPDFConvertLwc** Omniscript converts one or more previously generated `.docx` or `.pptx` documents into `.pdf` files in a single run. Expose it to users via the same custom-button pattern described in [Section 1](#1-create-a-custom-generate-document-button).

### 4.1 User Workflow

1. **Enter Object ID** — provide the Salesforce record ID (for example, a contract ID) the system uses to retrieve data.
2. **Generation Options** — accept defaults or override:
   - **Document Title** *(mandatory in DocGen 2.0)* — comma-separate titles to map to multiple documents.
   - **Docx Content Version Ids** — comma-separated `ContentVersion` IDs of the previously generated `.docx`/`.pptx` documents to convert. (IDs are obtainable from Developer Console.)
   - Use `--clear--` to leave a field blank.
3. **Generate Document** — converts and presents the PDF(s) for download per the chosen options.

### 4.2 Step Properties for fndmultiPDFConvertLwc

| Omniscript Step | Properties |
| --- | --- |
| **Script Configuration** | **Type:** `docGenerationSample` • **SubType:** `fndMultiPDFConvertLwc` |
| **EnterDetails** | **objectId** — Salesforce record ID; assigned to `contextID` in **Set Values**. |
| **GenerationOptions** | **DownloadFileFormat:** `PDF` (default) or `None`. **AttachFileFormat:** `PDF` (default) or `None`. **DocumentViewer:** `ClientSideViewer` (default) or `None` (disables preview). **DocumentTitle:** *(mandatory)* — comma-separated titles override file names and appear in preview. **docxContentVersionIds:** comma-separated `ContentVersion` IDs of source `.docx`/`.pptx` files. |
| **Set Generation Options** | Maps the values to: `attachFileFormat`, `contextId`, `documentTitle`, `documentViewer`, `docxContentVersionIds`, `downloadFileFormat`, `pdfGenerationSource`. |
| **Generate Document** | Uses the **clmOsMultiPdfDocumentConverter** LWC to generate and show the PDF, with download option. |

---

## 5. Configure Browser-Based Preview for Omnistudio Document Generation

The browser previewer renders PDF/DOCX/PPT formatting and visual elements directly in the browser.

### 5.1 Required Editions / Permissions

- Available in: **Lightning Experience**.
- Editions: **Professional**, **Enterprise**, **Unlimited**, **Developer**.
- Configuring previewer settings requires a **System Admin**.

### 5.2 Prerequisite

Set **PdfGenerationSource** to **Vlocity Client Side**. (See *Configure the PDF Conversion Custom Option* in Salesforce Help.)

### 5.3 Configure the Browser to Open PDFs

Example for **Firefox**:

1. **Settings → General**.
2. Under **Files and Applications → Applications**, locate **Portable Document Format (PDF)**.
3. Set the action to **Open in Firefox**.

> Exact steps differ per browser, but the goal is the same: the browser must open PDFs inline rather than forcing a download.

### 5.4 Add the Visualforce Domain to Remote Site Settings

1. Open any Omniscript and click **Preview**.
   - If you only use the document generation flow from the Contract record page, instead go to the **Details** tab → contract document section, right-click the **ContractDocumentNewDisplay** VF component, and complete remote site settings.
2. Right-click in the white space of the previewer and select **View Frame Source** to open the iframe in a new tab.
3. Copy the `vf.force.com` URL from the address bar (for example, `https://MyDomainName--c.vf.force.com`).
4. **Setup → Quick Find → Remote Site Settings**.
5. Click **New Remote Site** and enter:
   - **Remote Site Name:** any name without spaces (for example, `LWC_VF`).
   - **Remote Site URL:** the copied `vf.force.com` URL.
   - **Active:** ✅ enabled.
6. Save.

### 5.5 Reference

- *Supported PDF Features and Formatting Capabilities* — describes preview-supported PDF features.
- *Regenerate a File Preview* — re-renders a stored file's preview if missing or stale.

---

## 6. Quick-Reference: Custom Button URL Parameters

When wiring the fndSingleDocxLwc Omniscript into a custom button, these URL parameters drive its behavior:

| Parameter | Purpose | Example |
| --- | --- | --- |
| `c__target` | Omniscript wrapper target (already in the **How to Launch** URL). | `c:docGenerationSampleFndSingleDocxLwcEnglish` |
| `c__layout` | Layout mode. | `lightning` |
| `c__tabIcon` | Tab icon. | `custom:custom18` |
| `c__TemplateType` | Template type. | `MicrosoftWord` |
| `c__ObjectId` | Record ID — pass the merge field for the host object. | `{!WorkOrder.Id}` |
| `c__templateId` | Preselect a template (skips Pick-a-Template window when paired with the customizations in Section 2). | `2dt5f0000004C98AAE` |

---

## 7. Implementation Checklist

- [ ] Locate **fndSingleDocxLwc** in your org and copy its Lightning launch URL.
- [ ] Decide which host object(s) need a **Generate Document** button (Work Order, Opportunity, custom object, etc.).
- [ ] Create the **Detail Page Button** with `c__TemplateType` and `c__ObjectId={!Object.Id}` parameters.
- [ ] Add the button to the relevant page layouts.
- [ ] (Optional) Clone fndSingleDocxLwc, add the **Conditional View** rule on **PickTemplate**, update **SetValues**' `elementValueMap`, and update **SetGenerationOptions** to honor `c__templateId`. Append `c__templateId=<id>` to the button URL.
- [ ] (Optional) Repeat the button pattern for **fndmultiPDFConvertLwc** if users need to batch-convert previously generated DOCX/PPTX into PDF.
- [ ] Confirm `PdfGenerationSource = Vlocity Client Side`.
- [ ] Add the org's `vf.force.com` domain to **Remote Site Settings** for the browser-based previewer.
- [ ] Verify each user's browser is configured to open PDFs inline.
- [ ] Smoke-test: from a record, click **Generate Document** → confirm Object ID/Template Type prefill, template is preselected (if configured), generation options apply, and download/attach/email all behave as expected.
