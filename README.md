# Southern California Edison (southern-california-edison)

Southern California Edison (SCE) is the regulated electric utility subsidiary of Edison International, delivering power to roughly 15 million people across a 50,000 square-mile service territory in central, coastal, and southern California. In the United States energy value chain SCE sits at the distribution and retail layer as an investor-owned utility (IOU) regulated by the California Public Utilities Commission, operating the meters, the distribution grid, and the customer of record relationship that every downstream energy-data platform, DER aggregator, demand response provider, and solar installer ultimately depends on. Its API posture splits cleanly in two. Consumer data is mandated but closed to the open web — SCE runs Green Button Connect My Data through its Customer Data Access platform under CPUC tariff Rule 26 (Advice 3087-E, Decision 14-05-016), and states publicly that third parties need OAuth 2.0 and bulk API capability consistent with the NAESB ESPI standard, but publishes no developer portal, no base URI, no OpenAPI, and no sandbox — a third party must register with a Taxpayer Identification Number, accept terms, and pass a machine-to-machine connectivity test before any endpoint is disclosed. Grid data is genuinely open — SCE's Distribution Resources Plan External Portal (DRPEP) serves Integration Capacity Analysis, distribution circuit, PSPS, fire map, and load growth layers over an anonymous, unauthenticated ArcGIS REST service catalog that any developer can query today without a key.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/southern-california-edison/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/southern-california-edison/refs/heads/main/apis.yml)

## Tags

- Energy
- United States
- Utilities
- Electricity
- Smart Metering
- Green Button
- Grid
- Demand Response
- Solar
- DER
- EV Charging
- California

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### SCE Green Button Connect My Data

SCE's Customer Data Access (CDA) platform, through which a customer can authorize a registered third party to receive their electric usage and billing data. SCE states on its public third-party page that the program "leverages technology consistent with the standard adopted by the North American Energy Standards Board (NAESB) to support the Green Button Initiative" and that third parties must have "technical system capabilities to complete customer authorization using OAuth 2.0 and process files that are transferred using bulk API." No base URI, endpoint list, OpenAPI definition, scope list, or sandbox is published anywhere on sce.com — the interface is disclosed only after a third party registers, provides a Taxpayer Identification Number, accepts the Third Party Registration Terms & Conditions, and passes an SCE-run connectivity test. The obligation to offer it is a California regulatory one, carried in SCE tariff Rule 26 "Release of Customer Data or Energy Usage Related Data to Third Parties" (Advice 3087-E, filed 2014-07-30, effective 2014-08-29, implementing CPUC Decision 14-05-016), which explicitly ties technical eligibility criteria to the NAESB ESPI standard.

- **Human URL:** [https://www.sce.com/partners/3rd-party-energy-providers/access-energy-usage-data](https://www.sce.com/partners/3rd-party-energy-providers/access-energy-usage-data)
- **Base URL:** not published

#### Tags

- Green Button
- Customer Data
- Smart Metering
- OAuth
- ESPI

#### Properties

- [Documentation](https://www.sce.com/partners/3rd-party-energy-providers/access-energy-usage-data)
- [Portal](https://www.sce.com/partners/partnerships/thirdpartylandingpage)
- [Signup](https://www.sce.com/user-registration?userType=4)
- [Terms of Service](https://www.sce.com/terms-conditions/customer-data-access-terms-conditions)
- [Reference — Tariff Rule 26 (PDF)](https://www.sce.com/sites/default/files/inline-files/Rule26.pdf)
- [Reference — Data Element Descriptions (PDF)](https://www.sce.com/sites/default/files/inline-files/Data_Element_Descriptions_SCE_1.pdf)
- [Support](mailto:GreenButton@sce.com)

### SCE DRPEP ArcGIS REST Services

The machine-readable grid data layer behind SCE's Distribution Resources Plan External Portal (DRPEP). An anonymous, unauthenticated Esri ArcGIS REST service catalog (server version 11.1) exposing fifteen hosted FeatureServers covering Integration Capacity Analysis circuit segments and constraint layers, distribution circuits, Distribution Deferral Opportunity Report and Grid Needs Assessment layers, Locational Net Benefit Analysis, PSPS boundaries, fire map, subtransmission and available-load heat maps, and load growth penetration. SCE publishes no developer documentation for these endpoints, but they are live, key-free, and self-describing over the standard Esri ArcGIS REST interface, and a companion ArcGIS Hub open data site is served at drpep-sce2.opendata.arcgis.com. This is the one place SCE data is genuinely open to any developer without registration.

- **Human URL:** [https://drpep.sce.com/drpep/](https://drpep.sce.com/drpep/)
- **Base URL:** `https://drpep.sce.com/arcgis_server/rest/services`

#### Tags

- Grid
- Integration Capacity Analysis
- Open Data
- GIS
- Distributed Energy Resources

#### Properties

- [API Reference](https://drpep.sce.com/arcgis_server/rest/services/Hosted?f=json)
- [Reference](arcgis/sce-drpep-arcgis-services-catalog.json)
- [Reference](arcgis/sce-drpep-ica-layer-featureserver.json)
- [Reference](arcgis/sce-drpep-arcgis-rest-info.json)
- [Documentation](https://drpep.sce.com/drpep/)
- [Documentation](https://drpep-sce2.opendata.arcgis.com/)

## Common Properties

- [Website](https://www.sce.com/)
- [Portal](https://www.sce.com/partners/partnerships/thirdpartylandingpage)
- [Signup](https://www.sce.com/user-registration?userType=4)
- [Documentation — Access Energy Usage Data](https://www.sce.com/partners/3rd-party-energy-providers/access-energy-usage-data)
- [Documentation — Energy Data Reports & Compliances](https://www.sce.com/regulatory/regulatory-information/energy-data-reports-compliances)
- [Documentation — Rule 24 FAQs](https://www.sce.com/partners/3rd-party-energy-providers/rule-24-faqs)
- [Terms of Service](https://www.sce.com/terms-conditions/customer-data-access-terms-conditions)
- [Support](mailto:3RDPARTY@sce.com)
- [LinkedIn](https://www.linkedin.com/company/sce)

## Mandate and Access Posture

| Dimension | Finding |
| --- | --- |
| Mandate regime | `other` — California CPUC tariff Rule 26 (Advice 3087-E, Decision 14-05-016). Not CDR-energy, not Ontario Green Button, and not the voluntary US case: SCE is compelled by a state tariff. |
| Mandate status | `live-claimed-unverified` — the tariff was read directly and SCE states the program operates, but no endpoint, base URI, register entry, or standards-conformant surface could be observed anonymously. |
| Data standard | NAESB ESPI / Green Button (Connect My Data). No version published. |
| Consumer data API | Yes, but fully gated — no public contract of any kind. |
| Market/grid data open | Yes — anonymous ArcGIS REST catalog at `drpep.sce.com`, 15 FeatureServers, no key. |
| Access gate | `application-approval` — Third Party User ID, Taxpayer Identification Number, terms acceptance, then an SCE-run connectivity test. |
| Auth model | OAuth 2.0 plus a bulk file API for consumer data (specifics undisclosed); none for grid data. Okta fronts SCE web logins. |
| Developer portal | None. No `developer.`, `developers.`, `docs.`, `data.`, or `apis.` subdomain resolves; `api.sce.com` is an IBM API Connect gateway returning a 500 fault to every anonymous path. |
| OpenAPI published by SCE | None. SCE publishes no OpenAPI, Swagger, AsyncAPI, GraphQL or Postman contract anywhere. The spec in `openapi/` is an API Evangelist generation from live anonymous probes and is labelled as such. |

Full probe log with HTTP status for every URL tested is in [review.yml](review.yml).

## Harvested Artifacts (verbatim, as fetched)

- `arcgis/sce-drpep-arcgis-services-catalog.json` — the DRPEP hosted service catalog
- `arcgis/sce-drpep-arcgis-rest-info.json` — ArcGIS Server REST info document
- `arcgis/sce-drpep-*-featureserver.json` — all 15 hosted FeatureServer descriptors (47 feature layers, 11 tables)
- `arcgis/sce-drpep-hub-dcat-us-1.1.json` — DCAT-US 1.1 catalog feed from the paired ArcGIS Hub site (47 datasets)
- `well-known/southern-california-edison-okta-openid-configuration.json` — SCE Okta OIDC discovery document
- `well-known/southern-california-edison-okta-oauth-authorization-server.json` — SCE Okta RFC 8414 metadata

## Generated and Derived Artifacts

Produced by the API Evangelist enrichment pipeline from the harvested documents and live probes. None of these are
published by SCE; each carries provenance frontmatter.

- `openapi/southern-california-edison-drpep-arcgis-openapi.yml` — OpenAPI 3.1, 8 operations, 20 schemas, 58-entry layer inventory
- `overlays/southern-california-edison-drpep-arcgis-overlay.yaml` — the API Evangelist assessment layer over that spec
- `json-schema/southern-california-edison-drpep-layers-schema.json` — attribute schemas for 11 marquee layers/tables
- `data-model/southern-california-edison-data-model.yml` — 58 entities, 13 relationships, join keys
- `conventions/southern-california-edison-conventions.yml` — `f=json`, offset pagination, `exceededTransferLimit`, no rate-limit signalling
- `errors/southern-california-edison-error-codes.yml` — the ArcGIS error envelope returned with HTTP 200
- `authentication/southern-california-edison-authentication.yml` — none for DRPEP, OAuth 2.0 claimed-undisclosed for Green Button, Okta for logins
- `conformance/southern-california-edison-conformance.yml` — ArcGIS REST / GeoJSON / DCAT verified; ESPI and OAuth claimed only
- `lifecycle/southern-california-edison-lifecycle.yml` — no versioning, deprecation, SLA, changelog or API status page
- `security/southern-california-edison-domain-security.yml` — TLS 1.3 both hosts, HSTS on www only, SPF + DMARC quarantine, no DNSSEC/CAA
- `well-known/southern-california-edison-well-known.yml` — the full `/.well-known/` probe index
- `llms/southern-california-edison-llms.txt` — agent-facing summary of what is and is not callable
- `skills/` — four packaged agent skills, three grounded in real operationIds plus the Green Button registration path

## Maintainers

- Kin Lane — kin@apievangelist.com
