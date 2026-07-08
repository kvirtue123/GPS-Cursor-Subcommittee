---
name: sf-dependent-address-picklist
description: Read Salesforce compound-Address State/Country dependent picklists in Apex (via the validFor bitVector) and drive dependent lightning-combobox fields in an LWC, so users pick valid ISO codes instead of typing free text that throws DML errors. TRIGGER when an Address sub-field (Location__StateCode__s / __CountryCode__s) save throws a DML error, or when wiring country→state dependent comboboxes. DO NOT TRIGGER for plain (non-compound, non-picklist) text address fields.
---

# Salesforce Dependent Address Picklist (State/Country)

When State/Country Picklists are enabled, compound-Address sub-fields (`Location__StateCode__s`, `Location__CountryCode__s`) are an internal **dependent picklist**: state values are controlled by country. Free-text input ("Kenya" instead of "KE") throws a DML error on save. This skill reads the dependency map in Apex and renders dependent comboboxes in the LWC.

## When to use
- A compound `Address` field with State/Country Picklists enabled.
- You see DML errors saving a segment/record with a typed state/country.
- You need country→state dependent dropdowns sourced from org metadata.

## When NOT to use
- Plain text address fields (no picklist constraint) — just use `lightning-input`.
- Non-Address custom dependent picklists (same `validFor` technique applies, but the field describe differs).

## Dependencies & prerequisites
- State and Country/Territory Picklists enabled in the org.
- Apex: `Schema.DescribeFieldResult.getPicklistValues()`, `EncodingUtil` (base64/hex).
- LWC: `lightning-combobox`, `@wire` to a cacheable Apex method.

## The core mechanic
Each dependent `PicklistEntry` carries a base64 `validFor` bitVector encoding which controlling (country) indices it's valid for. Decode it to bytes, then for each country index check `byte[idx/8] & (1 << (7 - idx%8))`.

## Canonical example (Apex, abridged)
```apex
@AuraEnabled(cacheable=true)
public static Map<String, Object> getAddressPicklistValues() {
    Schema.DescribeFieldResult countryDesc =
        Travel_Itinerary_Segment__c.Location__CountryCode__s.getDescribe();
    Schema.DescribeFieldResult stateDesc =
        Travel_Itinerary_Segment__c.Location__StateCode__s.getDescribe();
    // build controllingIndexMap from countryDesc.getPicklistValues()
    // for each active state entry: decode validFor → bytes, test the bit per country idx
    // group statesByCountry[countryCode] = [{label, value}, …]
    // return { countries, statesByCountry }
}
private static List<Integer> decodeBase64ToBytes(String b64) {
    String hex = EncodingUtil.convertToHex(EncodingUtil.base64Decode(b64));
    List<Integer> bytes = new List<Integer>();
    for (Integer i = 0; i < hex.length(); i += 2)
        bytes.add(Integer.valueOf(hex.substring(i, i + 2), 16));
    return bytes;
}
```
LWC: wire the method, expose `countryOptions` + `stateOptions` getters (state filtered by selected country), and **clear state when country changes**. Country combobox first (dependency direction). Full step-by-step in `README.md`.

<!-- Provenance: project `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder). SKILL distilled from FIX-LOCATION-PICKLISTS.md. Catalog ID S6. -->
