---
name: Assess SCE grid needs, deferral opportunities and wildfire/PSPS exposure for a circuit
description: Combine Southern California Edison's Grid Needs Assessment, Distribution Deferral Opportunity Report, Locational Net Benefit Analysis, PSPS and CPUC fire-threat layers to profile a distribution circuit's deficiencies, DER deferral value and shutoff/fire exposure — all anonymously.
api: openapi/southern-california-edison-drpep-arcgis-openapi.yml
operations: [listHostedServices, getFeatureService, getFeatureLayer, queryFeatureLayer, getDrpepOperationalFlexibility]
generated: '2026-07-27'
method: generated
source: openapi/southern-california-edison-drpep-arcgis-openapi.yml + data-model/southern-california-edison-data-model.yml
---

# Grid needs, deferral value and wildfire exposure for an SCE circuit

Four of SCE's CPUC Distribution Resources Plan deliverables are published as anonymous ArcGIS layers on the same host,
keyed by the same circuit attributes. Together they answer: *does this circuit have a forecast deficiency, could a DER
defer the wire upgrade, and is the circuit inside a PSPS or fire-threat area?*

Base host `https://drpep.sce.com`; all calls are `GET` with `f=json`.

## 1. Grid Needs Assessment — is there a deficiency? (`queryFeatureLayer`)

```
GET /arcgis_server/rest/services/Hosted/GNA_Layer/FeatureServer/2/query
    ?where=circuit_name%3D%27<CIRCUIT>%27&outFields=*&returnGeometry=false&f=json
```

`GNA_Layer` layers: `0` GNA Substations, `1` GNA Subtransmission, `2` GNA - Circuits, plus tables `3` Circuit Level
Planning Assumptions, `4` GNA FGDB DATE TIME, `5` Substation Level Planning Assumptions. Look for `grid_need`,
`deficiency_year`, `operating_date` and the `d_y*` / `dp_y_max` / `dy_max` deficiency series. Join the planning
assumptions table on `circuit_name` (use `getFeatureLayer` on layer 3 first to read its field list).

## 2. Deferral opportunity — could a DER defer the upgrade?

```
GET .../Hosted/DDOR_Layer/FeatureServer/2/query
    ?where=circuit_name%3D%27<CIRCUIT>%27&outFields=*&returnGeometry=false&f=json
```

The Distribution Deferral Opportunity Report is scoped to needs identified in the GNA; both carry `gna_id`, which is
the reliable join key between them (`DDOR_Layer` layers: `0` Substations, `1` Subtransmission, `2` Circuits, table `3`
FGDB DATE TIME).

## 3. Locational value — what is avoided cost worth here?

```
GET .../Hosted/LNBA_Layer/FeatureServer/1/query   # 1 short-term, 2 mid-term, 3 long-term planning
    ?where=circuit_name%3D%27<CIRCUIT>%27&outFields=*&returnGeometry=false&f=json
```

`LNBA_Layer/0` carries the DER Growth Scenario. `maxRecordCount` on this service is 5,000.

## 4. Shutoff and fire exposure

```
GET .../Hosted/DRP_PSPS_Layer/FeatureServer/1/query      # PSPS_Circuit (0 = PSPS_Subtransmission)
    ?where=circuit_name%3D%27<CIRCUIT>%27&outFields=*&f=json
```

CPUC fire-threat tiers live in `Firemap`: `0` Tree Mortality Tier 1, `1` CPUC Fire Threat Tier 2 (Elevated), `2` Tier 3
(Extreme). These polygons carry **no circuit key** — relate them spatially by posting the circuit geometry as a
`geometry` + `geometryType=esriGeometryPolyline` + `spatialRel=esriSpatialRelIntersects` filter, not by attribute join.

## 5. Operational flexibility status (optional, unstable)

`getDrpepOperationalFlexibility` — `GET /api/op3?circuit=<CIRCUIT>&kv=<KV>` — is the DRPEP web app's own call. It
answers anonymously (`{"beingProcessed":false,"title":null,"op3Disclaimer":null}` for an unknown circuit) but SCE
publishes no contract for it. Treat it as best-effort; the durable Op Flex data is in the `ICA_Layer` op-flex layers
(2, 10, 13) and the `*_op_flex` attributes on layer 0.

## Rules

- Circuit identity is a **name string**, not an id — normalize case and whitespace, and confirm names against
  `Distribution_circuits/0` (`Distribution Circuits`) before trusting a no-match.
- Errors come back **HTTP 200** with an `error` body.
- Each service publishes its own refresh timestamp (`*FGDB DATE TIME` tables). GNA, DDOR and LNBA are filed on the CPUC
  DRP calendar, so their vintages differ from ICA — state the vintage of every layer you combine.
- The full 58-entity map and join keys: `data-model/southern-california-edison-data-model.yml`.
