<!-- Provenance: copied as-shipped from `ItineraryBuilder/FIX-LOCATION-PICKLISTS.md` (github.com/kvirtue/ItineraryBuilder). This is the full step-by-step backing the `sf-dependent-address-picklist` skill (catalog S6). Field names (Travel_Itinerary_Segment__c, etc.) are this project's specifics — generalize when reusing. -->

# FIX: Segment Panel Location Country & State — Text → Picklist

## Problem

The `segmentPanel` LWC uses `<lightning-input type="text">` for **Location (State)** and **Location (Country)**. The underlying Salesforce object `Travel_Itinerary_Segment__c` uses a **compound Address field** (`Location__c`) whose sub-fields `Location__StateCode__s` and `Location__CountryCode__s` are **picklist-constrained** (State/Country Picklists enabled in the org). Free-text values that don't match valid ISO picklist entries cause **DML exceptions** in `ItineraryBuilderController.saveSegment()`.

## Root Cause

In `segmentPanel.html` (lines ~71–87), both fields are plain text inputs:

```html
<!-- CURRENT — BROKEN -->
<lightning-input type="text" label="Location (State)" value={_state} ...></lightning-input>
<lightning-input type="text" label="Location (Country)" value={_country} ...></lightning-input>
```

The Apex controller assigns these values directly to the compound address sub-fields:

```apex
// ItineraryBuilderController.cls — saveSegment()
seg.Location__StateCode__s = stateCode;   // expects valid picklist code
seg.Location__CountryCode__s = countryCode; // expects valid picklist code
```

When the user types free-form text like "Kenya" instead of the picklist code "KE", the upsert throws a DML error.

## Solution

Replace both `<lightning-input type="text">` fields with `<lightning-combobox>` components that source their options from the org's Address State/Country picklist metadata. State must be dependent on Country.

---

## Files to Modify

| File | Change |
|------|--------|
| `force-app/main/default/lwc/segmentPanel/segmentPanel.html` | Replace 2x `lightning-input` with `lightning-combobox` |
| `force-app/main/default/lwc/segmentPanel/segmentPanel.js` | Add Apex wire for picklist values, add country/state option getters, update change handlers |
| `force-app/main/default/classes/ItineraryBuilderController.cls` | Add new `@AuraEnabled(cacheable=true)` method to return picklist values |

No changes needed to: `_buildSegmentData()`, `saveSegment()` Apex method, `itineraryBuilder.js` parent, or any other components.

---

## Step 1 — Apex: Add Picklist Metadata Method to ItineraryBuilderController.cls

Add these two methods to the class:

```apex
@AuraEnabled(cacheable=true)
public static Map<String, Object> getAddressPicklistValues() {
    Map<String, Object> result = new Map<String, Object>();

    // Get Country picklist values from Address compound field
    Schema.DescribeFieldResult countryDesc =
        Travel_Itinerary_Segment__c.Location__CountryCode__s.getDescribe();
    List<Map<String, String>> countries = new List<Map<String, String>>();
    for (Schema.PicklistEntry pe : countryDesc.getPicklistValues()) {
        if (pe.isActive()) {
            countries.add(new Map<String, String>{
                'label' => pe.getLabel(),
                'value' => pe.getValue()
            });
        }
    }
    result.put('countries', countries);

    // Get State picklist values, grouped by controlling country code
    Schema.DescribeFieldResult stateDesc =
        Travel_Itinerary_Segment__c.Location__StateCode__s.getDescribe();

    // Build controlling value index map
    List<Schema.PicklistEntry> controllingEntries = countryDesc.getPicklistValues();
    Map<String, Integer> controllingIndexMap = new Map<String, Integer>();
    for (Integer i = 0; i < controllingEntries.size(); i++) {
        controllingIndexMap.put(controllingEntries[i].getValue(), i);
    }

    // Parse dependent picklist bitVector for each state entry
    Map<String, List<Map<String, String>>> statesByCountry =
        new Map<String, List<Map<String, String>>>();

    for (Schema.PicklistEntry pe : stateDesc.getPicklistValues()) {
        if (!pe.isActive()) continue;
        String validForBase64 = ((Map<String, Object>)
            JSON.deserializeUntyped(JSON.serialize(pe))).get('validFor')?.toString();

        if (validForBase64 == null) continue;

        List<Integer> validForBytes = decodeBase64ToBytes(validForBase64);

        for (String countryCode : controllingIndexMap.keySet()) {
            Integer idx = controllingIndexMap.get(countryCode);
            Integer byteIndex = idx / 8;
            Integer bitIndex = 7 - Math.mod(idx, 8);
            if (byteIndex < validForBytes.size() &&
                (validForBytes[byteIndex] & (1 << bitIndex)) != 0) {
                if (!statesByCountry.containsKey(countryCode)) {
                    statesByCountry.put(countryCode,
                        new List<Map<String, String>>());
                }
                statesByCountry.get(countryCode).add(new Map<String, String>{
                    'label' => pe.getLabel(),
                    'value' => pe.getValue()
                });
            }
        }
    }
    result.put('statesByCountry', statesByCountry);

    return result;
}

private static List<Integer> decodeBase64ToBytes(String base64String) {
    Blob decoded = EncodingUtil.base64Decode(base64String);
    String hex = EncodingUtil.convertToHex(decoded);
    List<Integer> bytes = new List<Integer>();
    for (Integer i = 0; i < hex.length(); i += 2) {
        bytes.add(Integer.valueOf(hex.substring(i, i + 2), 16));
    }
    return bytes;
}
```

### Why this approach

Salesforce compound Address fields with State/Country Picklists enabled use a **dependent picklist** pattern internally. `Location__StateCode__s` values are controlled by `Location__CountryCode__s`. The `validFor` bitVector on each `PicklistEntry` encodes which controlling values (countries) each dependent value (state) is valid for. This is the standard Apex pattern for reading dependent picklist metadata.

---

## Step 2 — LWC JS: segmentPanel.js Changes

### 2a. Add import at top of file

```js
import getAddressPicklistValues from '@salesforce/apex/ItineraryBuilderController.getAddressPicklistValues';
```

Also add `wire` to the lwc import if not already there:

```js
import { api, wire, LightningElement } from 'lwc';
```

### 2b. Add state properties (after existing property declarations ~line 48)

```js
_countryOptions = [];
_statesByCountry = {};
```

### 2c. Add wire adapter (after existing properties, before getters)

```js
@wire(getAddressPicklistValues)
wiredPicklists({ data, error }) {
    if (data) {
        this._countryOptions = data.countries.map(c => ({
            label: c.label,
            value: c.value
        }));
        this._statesByCountry = data.statesByCountry || {};
    } else if (error) {
        console.error('Failed to load address picklists', error);
    }
}
```

### 2d. Add computed getters (in the getters section)

```js
get countryOptions() {
    return this._countryOptions;
}

get stateOptions() {
    if (!this._country || !this._statesByCountry[this._country]) {
        return [];
    }
    return this._statesByCountry[this._country].map(s => ({
        label: s.label,
        value: s.value
    }));
}

get isStateDisabled() {
    return this.isReadOnly || this.stateOptions.length === 0;
}
```

### 2e. Update handleCountryChange to clear state on country change

Replace existing `handleCountryChange`:

```js
handleCountryChange(event) {
    this._country = event.detail.value;
    // Clear state when country changes — dependent picklist
    this._state = '';
    this._debouncedLocationSave();
}
```

`handleStateChange` stays the same — no modification needed.

---

## Step 3 — LWC HTML: segmentPanel.html Changes

### Find and replace this block (the State/Country grid, ~lines 71–87):

**REMOVE:**

```html
<div class="slds-grid slds-gutters_small slds-wrap">
    <div class="slds-col slds-size_1-of-2">
        <lightning-input
            type="text"
            label="Location (State)"
            value={_state}
            disabled={isReadOnly}
            onchange={handleStateChange}
        ></lightning-input>
    </div>
    <div class="slds-col slds-size_1-of-2">
        <lightning-input
            type="text"
            label="Location (Country)"
            value={_country}
            disabled={isReadOnly}
            onchange={handleCountryChange}
        ></lightning-input>
    </div>
</div>
```

**REPLACE WITH:**

```html
<div class="slds-grid slds-gutters_small slds-wrap">
    <div class="slds-col slds-size_1-of-2">
        <lightning-combobox
            label="Location (Country)"
            value={_country}
            options={countryOptions}
            disabled={isReadOnly}
            onchange={handleCountryChange}
            placeholder="Select Country..."
        ></lightning-combobox>
    </div>
    <div class="slds-col slds-size_1-of-2">
        <lightning-combobox
            label="Location (State)"
            value={_state}
            options={stateOptions}
            disabled={isStateDisabled}
            onchange={handleStateChange}
            placeholder="Select State..."
        ></lightning-combobox>
    </div>
</div>
```

**Note:** Country is now FIRST (left column) so the user selects country before state. This matches the dependency direction.

---

## What Does NOT Change

- `_buildSegmentData()` in `segmentPanel.js` — already sends `stateCode` and `countryCode` correctly
- `saveSegment()` in Apex — already assigns to `Location__StateCode__s` and `Location__CountryCode__s`
- `_syncFromSegment()` — already reads from `Location__StateCode__s` and `Location__CountryCode__s`
- `itineraryBuilder.js` parent — no changes needed
- `handleSegmentTypedDropped` / `handleSegmentResized` — already pass empty strings or existing values

---

## Behavior Summary

- **Country must be selected first** — State combobox is disabled and empty until a country is picked
- **Changing country clears state** — Prevents stale state codes that don't belong to the new country
- **Values are ISO codes** (e.g., "US", "KE", "CA" for countries; "VA", "CA", "NY" for US states) — matching what the compound Address field expects
- **Debounced save still applies** — 500ms `_debouncedLocationSave()` works unchanged
- **Existing segments load correctly** — `_syncFromSegment()` reads the stored codes, comboboxes display matching labels

---

## Testing Checklist

1. [ ] Country combobox loads with all active countries from org metadata
2. [ ] Selecting a country populates state combobox with dependent states
3. [ ] Changing country clears previously selected state
4. [ ] State combobox disabled when no country selected
5. [ ] Saving a segment with valid country/state codes succeeds (no DML error)
6. [ ] Existing segments with saved country/state codes display correctly on load
7. [ ] Read-only mode disables both comboboxes
8. [ ] Smart Suggestions still work (they compare `Location__CountryCode__s` values like "KE")
9. [ ] New segment drag-and-drop works (empty country/state defaults)

---

## Deploy

```bash
sf project deploy start --source-dir force-app --target-org kvirtue@state.demo
```
