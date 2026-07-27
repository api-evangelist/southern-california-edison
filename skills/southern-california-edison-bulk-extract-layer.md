---
name: Bulk-extract a DRPEP layer from SCE
description: Pull a complete Southern California Edison DRPEP feature layer or table (up to hundreds of thousands of features) out of the anonymous ArcGIS REST API without tripping the transfer limit, or take the bulk file instead.
api: openapi/southern-california-edison-drpep-arcgis-openapi.yml
operations: [listHostedServices, getFeatureService, getFeatureLayer, queryFeatureLayer, downloadDrpepNodeCsv]
generated: '2026-07-27'
method: generated
source: openapi/southern-california-edison-drpep-arcgis-openapi.yml + conventions/southern-california-edison-conventions.yml
---

# Bulk-extract a DRPEP layer

The 15 SCE DRPEP hosted feature services answer anonymously, but every layer caps a single response at its
`maxRecordCount` (2,000 on most services; 5,000 on `AVL_LOAD_HEAT_MAP_TOGGLE`, `DRP_PSPS_Layer`, `LNBA_Layer` and
`MGEP_AVL_LOAD_HEAT_MAP`). `ICA_Layer` layer 0 alone holds 676,467 features, so a naive pull silently returns 2,000
rows and a truthful-but-easy-to-miss `"exceededTransferLimit": true`.

## Step 1 — size the job (`queryFeatureLayer`)

```
GET https://drpep.sce.com/arcgis_server/rest/services/Hosted/{serviceName}/FeatureServer/{layerId}/query
    ?where=1%3D1&returnCountOnly=true&f=json
```

Verified against `ICA_Layer/0`: `{"count":676467}`.

## Step 2 — read the layer contract (`getFeatureService`, `getFeatureLayer`)

```
GET .../Hosted/{serviceName}/FeatureServer?f=json          # maxRecordCount, capabilities, layer list
GET .../Hosted/{serviceName}/FeatureServer/{layerId}?f=json # fields, objectIdField, advancedQueryCapabilities
```

Confirm `advancedQueryCapabilities.supportsPagination` is true (it is on these layers) and note the object-id field
(`objectid`).

## Step 3 — page it

Offset paging, the simple path:

```
GET .../query?where=1%3D1&outFields=*&returnGeometry=false
    &orderByFields=objectid&resultOffset={n}&resultRecordCount=2000&f=json
```

Increment `resultOffset` by the page size until a response comes back with fewer rows than requested and without
`exceededTransferLimit`. Always set `orderByFields=objectid` — offset paging without a stable sort can repeat or skip rows.

Id-batching, the robust path for very large layers:

```
GET .../query?where=1%3D1&returnIdsOnly=true&f=json          # -> objectIds array
GET .../query?objectIds=1,2,3,...,2000&outFields=*&f=json    # <= maxRecordCount ids per call
```

Ask for `returnGeometry=false` whenever you only need attributes — geometry dominates payload size on the polyline
circuit layers. Use `f=geojson` when the destination is a GIS tool.

## Step 4 — consider not using the API at all

Two published bulk paths are cheaper than paging:

- **Per-segment CSV**: ICA feature layers carry `download_link`, `download_link_microgrid`, `download_link_forecast`
  and `download_link_microgrid_forecast` attributes that point directly at SCE's exports. The DRPEP app's own
  `downloadDrpepNodeCsv` endpoint (`GET /api/DownloadCSV?circuitName=&nodeName=`) is undocumented and returns
  `{"status":false}` for unknown inputs — prefer the attribute links.
- **Whole datasets**: the paired ArcGIS Hub site at `https://drpep-sce2.opendata.arcgis.com` publishes the same 47
  datasets as CSV, GeoJSON, KML and Shapefile, with a DCAT-US 1.1 catalog feed at `/api/feed/dcat-us/1.1.json`
  (saved in this repo at `arcgis/sce-drpep-hub-dcat-us-1.1.json`).

## Rules

- Failures return **HTTP 200** with an `error` object; branch on the body, never on the status line.
- No published rate limit, no `Retry-After`, no `RateLimit-*` headers. Serialize pages, back off on any anomaly.
- Record the data vintage with the extract: `ICA_Tables/0` (`DRPEP_FGDB_DATE_TIME`), `DDOR_Layer/3`, `GNA_Layer/4`,
  `LNBA_Layer/4`. There is no changelog or notification when SCE republishes.
- Attribute schemas for the marquee layers: `json-schema/southern-california-edison-drpep-layers-schema.json`.
