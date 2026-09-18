# Universal Data Layer Converter (Server) — by MD Niamul

A Google Tag Manager **Variable Template** for the **Server-side (sGTM)
container**. It reads the incoming server event data — whatever structure
it arrives in (GA4-style, Universal Analytics-style, a custom e-commerce
schema, a lead-gen payload, or anything else) — and converts it into the
exact payload shape a specific ad platform's Conversions API expects,
including SHA-256 hashing of personal data in the format each platform
requires.

No value is ever invented. A field that isn't present in the source stays
empty, or is returned as a placeholder you configure.

For the **client-side (Web container)** counterpart — which never hashes
data in the browser — see
[Universal Data Layer Converter (Web)](https://github.com/mdniamul0/gtm-universal-datalayer-converter-web).

## What it does

1. Reads the server container's full incoming event data via
   `getAllEventData()` ("Auto" mode), or reads a single object from another
   variable ("Variable" mode).
2. Auto-detects the canonical fields inside that object regardless of the
   exact key names used, via a large alias table and shallowest-match
   resolution.
3. Formats the result for the platform you choose: GA4, Google Ads, Meta
   Conversions API, TikTok Events API, LinkedIn Conversions API, Snapchat
   CAPI, Pinterest Conversions API, Microsoft Advertising (Bing UET), X
   (Twitter) Ads, Reddit, Quora, Outbrain, Taboola, a generic structure, or
   a Stape pass-through shape — or leave it as a platform-independent
   "Normalized" object.
4. **Hashes personal data (email, phone, name, address) with SHA-256**
   before it leaves the variable, normalized to each platform's required
   format:
   - **E.164 with a leading `+`** for Google Ads and Microsoft/Bing UET
     enhanced conversions (both require this exact shape).
   - **Digits-only, no `+`** for Meta, TikTok, Snapchat, Pinterest, and the
     other CAPI-style destinations.
   - Values that are already a 64-character hex string are detected and
     passed through unchanged — never re-hashed or reformatted.
   - Phone numbers must already include their country code in the source
     data; the template normalizes format but never invents a country code.
5. Optionally lets you override any single field with a manual path when
   auto-detection picks the wrong value.

## Parameters

| Parameter | Purpose |
|---|---|
| Data source | Read all incoming event data automatically, or point at another variable that already returns an object. |
| Output format (platform) | Which platform's payload shape to produce. |
| Return | Full object, just the event name, a single field by path, or the object as a JSON string. |
| Field path | Dot-notation path used with "Single field" return mode. |
| When a value is not found | Omit the key, return an empty string, or return a placeholder — the template never invents data. |
| Advanced → Manual mapping | Per-field overrides (`field` → source `path`) that win over automatic detection. |
| Advanced → Calculate value from items | If no top-level order value is found, sum `price × quantity` across items and mark `value_source: 'calculated_from_items'`. |
| Advanced → Enable debug logging | Logs the platform, the normalized model, and the output via `logToConsole` (scoped to the GTM debug/preview environment only). |
| Action source / Event conversion type | Passed through to platforms that need them. |
| SHA-256 hash email, phone and name fields | On by default. Turning this off still **normalizes** the data (trim, lowercase, phone formatted per platform) but skips the final hash step — only useful if a downstream system will hash it before it reaches the ad platform. |

## Security & privacy notes

- **Personal data is hashed by default** before it leaves the variable.
  Disabling the hash checkbox does not send raw, unformatted PII — values
  are still normalized — but you take on responsibility for hashing before
  the data reaches the ad platform's servers if you turn it off.
- **Permission:** this template requests `read_event_data` with
  `eventDataAccess: "any"`. This is required because, in Auto mode, it
  must accept whatever event structure your server container receives
  without knowing its shape in advance — that's the template's entire
  purpose. It does not forward, persist, or log this data anywhere beyond
  the debug-console output described below.
- **Permission:** `logging`, scoped to the `debug` environment only —
  nothing is logged in a live/production container.

## Testing

The template ships with unit tests under `___TESTS___` (run automatically
by Google's template validator) covering: GA4→Meta mapping with hashing,
normalized-but-unhashed behavior when hashing is off, the Google Ads
E.164-vs-Meta digits-only phone hash distinction, a custom schema→Snapchat
mapping, event-name-only output, empty-event-data behavior, and numeric
ID-to-string coercion. All scenarios pass. The template was additionally
verified with an independent Node.js test harness that mocks GTM's
sandboxed JS APIs (including `sha256Sync`) and exercises adversarial edge
cases — hash format per platform, already-hashed-value pass-through,
missing-PII handling, and the shared mapping engine's edge cases — beyond
the built-in suite.

## Installation

1. In your GTM Server container, go to **Templates → Variable Templates →
   Search Gallery**, and search for "Universal Data Layer Converter
   (Server) by MD Niamul" (once approved), **or**
2. Import `template.tpl` manually: **Templates → New → ⋮ → Import**.
3. Create a new variable from the template, choose your platform and
   options, and use it wherever you'd build a Conversions API payload by
   hand.

## About the author

Built by **MD Niamul**, founder of Digital Soldier Agency — a conversion
tracking, server-side tagging (sGTM), and web analytics specialist. 10×
Meta Blueprint Certified, 8× Google Ads & Microsoft Ads Certified, HubSpot
Certified Inbound Marketer, and an official Stape partner.

- Website: https://mdniamul.com
- LinkedIn: https://www.linkedin.com/in/mdniamul/

## License

Apache License 2.0 — see [LICENSE](./LICENSE).
