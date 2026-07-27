---
name: Query SCE hosting capacity (ICA) for a circuit or location
description: Find the Integration Capacity Analysis generation and load hosting capacity Southern California Edison publishes for a named distribution circuit, a node/segment, or an area of interest, using the anonymous DRPEP ArcGIS REST API.
api: openapi/southern-california-edison-drpep-arcgis-openapi.yml
operations: [listHostedServices, getFeatureService, getFeatureLayer, queryFeatureLayer]
generated: '2026-07-27'
method: generated
source: openapi/southern-california-edison-drpep-arcgis-openapi.yml + conventions/southern-california-edison-conventions.yml
---

# Query SCE hosting capacity (ICA)

Southern California Edison publishes Integration Capacity Analysis (ICA) — how much distributed generation or load a
circuit segment can absorb without upgrades — through the Distribution Resources Plan External Portal (DRPEP). The
interface is anonymous: no API key, no registration, no terms click-through.

Base host: `https://drpep.sce.com`. Every call is a `GET`. Always send `f=json` (or `f=geojson`); without it the
service returns an HTML directory page.

## 1. Confirm the layer you want (`listHostedServices`, `getFeatureService`, `getFeatureLayer`)

```
GET /arcgis_server/rest/services/Hosted?f=json
GET /arcgis_server/rest/services/Hosted/ICA_Layer/FeatureServer?f=json
GET /arcgis_server/rest/services/Hosted/ICA_Layer/FeatureServer/0?f=json
```

`ICA_Layer` layer `0` is **ICA - Circuit Segments**, the primary layer (676,467 features as of 2026-07-27). The layer
descriptor is the only field dictionary SCE provides — read it before writing a `where` clause. The key attributes:

| Field | Meaning |
|---|---|
| `circuit_name`, `circuit_voltage` | Distribution circuit and its voltage |
| `substation_name`, `substation_voltage`, `system_name` | Upstream substation and planning system |
| `node_id`, `section_id`, `phase` | Segment address on the circuit |
| `ica_overall_pv` | Photovoltaic static-grid hosting capacity |
| `uniform_generation`, `uniform_generation_op_flex` | Uniform generation static grid / operational flexibility |
| `ica_overall_load` | Uniform load capacity |
| `most_limiting_criteria_*` | Which constraint binds (thermal, steady-state voltage, voltage variation, protection, op flex) |
| `download_link*` | Per-segment CSV exports |
| `changed_date`, `changed_date_forecast` | Data vintage for that segment |

## 2. Query by circuit name (`queryFeatureLayer`)

```
GET /arcgis_server/rest/services/Hosted/ICA_Layer/FeatureServer/0/query
    ?where=circuit_name%3D%27Acapulco%27
    &outFields=circuit_name,node_id,section_id,ica_overall_pv,uniform_generation,most_limiting_criteria_pv_sg,changed_date
    &returnGeometry=false
    &f=json
```

Circuit names are plain strings (`'Acapulco'`) — quote them with single quotes inside the SQL-92 `where` clause.
To discover valid names first: `?where=1%3D1&outFields=circuit_name&returnDistinctValues=true&returnGeometry=false&f=json`.

## 3. Query by location instead of by name

Supply an envelope in WGS84 and let the service do the spatial filter:

```
GET .../ICA_Layer/FeatureServer/0/query
    ?geometry=-118.30,33.95,-118.10,34.10
    &geometryType=esriGeometryEnvelope
    &spatialRel=esriSpatialRelIntersects
    &inSR=4326&outSR=4326
    &outFields=circuit_name,ica_overall_pv,ica_overall_load
    &f=geojson
```

## 4. Read the binding constraint

When a segment's hosting capacity is low, the reason is in the `most_limiting_criteria_*` fields, and the detail sits
in the sibling layers of the same service — all queried with the same `queryFeatureLayer` operation:

| Layer id | Constraint |
|---|---|
| 2 / 13 | Generation and Uniform Generation Op Flex |
| 3 | Generation Protection |
| 4 / 7 | Generation and Load Steady State Voltage |
| 5 / 8 | Generation and Load Thermal |
| 6 / 9 | Generation and Load Voltage Variation |
| 11 / 14 / 15 | Solar PV, Uniform Generation and Uniform Load Static Grid |
| 12 | Unavailable Circuit Segments (no ICA published) |

Join them to layer 0 on `circuit_name` + `node_id` + `section_id`.

## Rules that will bite you

- **Errors arrive with HTTP 200.** A bad `where` clause returns `{"error":{"code":500,"message":"Syntax error []."}}`
  with a 200 status. Always test for an `error` key in the body — see `errors/southern-california-edison-error-codes.yml`.
- **Page everything.** `maxRecordCount` is 2,000 on this service. When a response carries
  `"exceededTransferLimit": true`, request the next page with `resultOffset` / `resultRecordCount`, or pull
  `returnIdsOnly=true` first and fetch in `objectIds` batches.
- **No quota is published, and that is not permission.** No rate-limit headers, no documented limit. Keep concurrency modest.
- **No version pin, no status page, no SLA.** The server is ArcGIS Enterprise 11.1 today and can change without notice
  (`lifecycle/southern-california-edison-lifecycle.yml`). Check the `ICA_Tables/0` (`DRPEP_FGDB_DATE_TIME`) table for
  the current data vintage before publishing an analysis.
- **Do not touch the `Utilities` folder.** It returns `{"error":{"code":499,"message":"Token Required"}}` and there is
  no self-service token.
