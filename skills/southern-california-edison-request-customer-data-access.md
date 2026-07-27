---
name: Obtain SCE customer energy data as a third party (Green Button Connect My Data)
description: >-
  The real, gated path to Southern California Edison customer usage and billing data — what SCE publishes, what it
  does not, and the four registration steps required before any endpoint is disclosed. Procedural, not callable —
  there is no public API surface to invoke.
api: null
operations: []
generated: '2026-07-27'
method: generated
source: >-
  https://www.sce.com/partners/3rd-party-energy-providers/access-energy-usage-data +
  https://www.sce.com/partners/partnerships/thirdpartylandingpage + Rule 26 tariff (Advice 3087-E)
---

# Getting SCE customer energy data as a third party

**There is no public API to call.** Do not attempt to construct ESPI or OAuth URLs against `api.sce.com` — the gateway
is live (IBM DataPower / API Connect behind Imperva) but every ESPI-shaped path returns HTTP 500 with
`Dynamic backend host not specified`. SCE discloses the interface only after registration. Anything an agent "finds"
that looks like an SCE Green Button endpoint on the open web is not from SCE.

## What SCE actually publishes

- The program exists and is called **Green Button Connect My Data**, running on SCE's **Customer Data Access (CDA)** platform.
- It "leverages technology consistent with the standard adopted by the North American Energy Standards Board (NAESB) to
  support the Green Button Initiative."
- A third party must have "technical system capabilities to complete customer authorization using **OAuth 2.0** and
  process files that are transferred using **bulk API**."
- The obligation is a California state tariff: **SCE Rule 26**, "Release of Customer Data or Energy Usage Related Data
  to Third Parties" (Advice 3087-E, filed 2014-07-30, effective 2014-08-29, implementing CPUC Decision 14-05-016) —
  <https://www.sce.com/sites/default/files/inline-files/Rule26.pdf>.

## What SCE does not publish

No base URI, no endpoint list, no ApplicationInformation document, no scope syntax, no token lifetime, no OpenAPI, no
sandbox, no developer portal, no error reference, no SLA. No Green Button Alliance certification entry names SCE.

## The four registration steps (verbatim from SCE's third-party landing page)

1. Create a **Third Party User ID** and password — the User ID must be an email address not previously registered as an
   SCE.com user ID. Entry point: <https://www.sce.com/user-registration?userType=4>
2. Supply the organization's **Taxpayer Identification Number (TIN)**.
3. Agree to SCE's **Third Party Registration Terms & Conditions** for the use of customer data —
   <https://www.sce.com/terms-conditions/customer-data-access-terms-conditions>
4. Pass a **connectivity test**: "a connectivity test, which tests the connection between your computer system and ours,
   must be performed. We'll contact a designated person familiar with the computer systems in your organization."

Only then is the machine-to-machine interface described to you.

## After registration: per-customer authorization

Each customer must separately authorize your organization through the **Green Button Connect My Data Authorization**
page inside SCE My Account. The non-API alternative is a **Customer Information Service Request (CISR)** form
(`ELECTRIC_FORMS_14-796.pdf`), which yields files rather than a feed.

## Choosing the right lane

| Lane | Who it is for | Contact |
|---|---|---|
| Green Button Connect My Data (Rule 26) | Any third party acting for a consenting customer | GreenButton@sce.com |
| Demand Response (CPUC Rule 24) | Demand response providers; 36 months of historical meter data | Rule24Support@sce.com |
| Energy Data Request Program (EDRP) | Government, academic and research requesters | EnergyDataRequest_EDRP@sce.com |
| General third-party support | Registration and onboarding questions | 3RDPARTY@sce.com |

## If you only need aggregate or grid data, skip all of this

- **Aggregated consumption**: quarterly Electric Usage By ZIP spreadsheets at
  <https://www.sce.com/regulatory/regulatory-information/energy-data-reports-compliances>
- **Grid data**: the DRPEP ArcGIS REST API is anonymous and needs none of the above — see
  `skills/southern-california-edison-query-hosting-capacity.md`.
