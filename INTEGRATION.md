# SafetyScan Clip-Lok International Demo v1.0 — Integrationsdokumentation

Status: **klargjort, ikke integreret.** Der er ingen backend, ingen API-URL, ingen API-nøgle, ingen Supabase, ingen cloud-database og ingen netværkskald. Al data ligger i browserens `localStorage` (nøglepræfiks `sscli_v1_`). Den konkrete integration aftales senere med Clip-Loks IT.

## 1. Funktioner

| Funktion | Beskrivelse |
|---|---|
| `exportReportAsJSON(reportId)` | Returnerer en ren, sproguafhængig JSON-struktur (schema 1.0). Kun interne koder — aldrig oversatte værdier. Tager også et rapport-objekt. |
| `sendToExternalSystem(report)` | Placeholder. Sender **intet**. Returnerer altid `{ "success": false, "mode": "demo", "message": "External integration not configured" }` (+ `report_id`). |
| `normalizeReport(r)` | Gør ældre rapporter (uden `schema_version`, med danske værdier) sproguafhængige ved læsning. |

## 2. Standard datamodel (schema_version "1.0")

| Felt | Type | Indhold / eksempel |
|---|---|---|
| report_id | string | `CL-2026-0001` |
| schema_version | string | `"1.0"` |
| source_system | string | `"safetyscan-clip-lok"` |
| company_id | string | `demo-company` (neutral demo-værdi) |
| site_id | string | `demo-site-01` |
| department_id | string\|null | afdeling fra lokationens stamdata, fx `warehouse` |
| object_id | string\|null | konkret objekt-ID (klar til QR, fx `CL-00482`) |
| object_type | string | objektkode, se §3 |
| qr_code | string\|null | QR-værdi (klar — ingen QR-administration endnu) |
| event_type | string | hændelseskode, se §3 |
| priority | string | `low` · `medium` · `high` · `critical` |
| status | string | `new` · `assigned` · `in_progress` · `resolved` · `closed` |
| description | string | brugerens fritekst (oversættes ikke) |
| location | string | intern lokations-ID, fx `rack-a02` |
| location_name | string | stamdatanavn (reference, ikke oversættelse) |
| latitude / longitude / accuracy | number\|null | kun hvis brugeren valgte GPS |
| location_source | string | `gps` · `manual` |
| event_at | ISO 8601 UTC | hændelsestidspunkt (kan ændres manuelt) |
| created_at | ISO 8601 UTC | oprettelsestidspunkt |
| updated_at / closed_at | ISO 8601 \| null | ekstra |
| reporter_id | string\|null | `null` i demo (intet login) |
| reporter_type | string | `named` · `anonymous` · `unspecified` (ekstra) |
| responsible_id | string\|null | funktionskode, se §3, eller masterdata-ID |
| supplier_id | string\|null | leverandør-ID når sagen er sendt til leverandør |
| recommended_action | string[] | handlingskoder, se §3 (AI-forslag accepteret af bruger) |
| images | object[] | `image_id`, `filename`, `mime_type`, `role` (`before`/`after`), `storage_reference` |
| attachments | array | tom i demo |
| history | object[] | audit trail: `at`, `action`, `from`, `to`, `by` (ekstra) |
| external_reference | string\|null | ID i kundens system efter fremtidig overførsel |
| integration_status | string | `not_sent` i demo (fremtid: `pending` · `sent` · `failed`) |

Billeder ligger lokalt som komprimeret JPEG i browseren. `storage_reference` er en lokal reference (`local-browser://CL-2026-0001/photo-1`). Der er ikke bygget ekstern storage.

Eksempel: se `example-payload.json`.

## 3. Interne koder

**Status:** `new` (Åben) · `assigned` (Tildelt) · `in_progress` (I gang) · `resolved` (Løst) · `closed` (Lukket)
Dashboard: *Åbne* = `new` · *Under behandling* = `assigned` + `in_progress` + `resolved` · *Lukkede* = `closed` · *Kritiske* = `critical` og ikke lukket.

**Prioritet:** `low` (grøn) · `medium` (gul) · `high` (orange) · `critical` (rød)

**Hændelsestype (event_type):** `safety` · `defect` · `near_miss` · `improvement` · `observation`

**Objekttype (object_type):** `rack` · `truck` · `autostore` · `miniload` · `baler` · `gate` · `ramp` · `battery_room` · `fire_equip` · `other`

**Ansvarlig funktion (responsible_id):** `warehouse_manager` · `team_leader` · `technical` · `maintenance` · `production` · `hse` · `safety_officer` · `external_supplier` (fritekst-navne gemmes som de er)

**Handlinger (recommended_action):** `cordon_area` · `contact_now` · `service_order_now` · `cordon_if_needed` · `contact_dept` · `follow_up_24h` · `plan_repair` · `inform_responsible` · `note_maintenance`

**Historik (history.action):** `created` · `status_changed` · `assigned` · `comment` · `after_photo` · `closed` · `sent_supplier`

**Lokationer (seed):** `rack-a01…rack-c01`, `automat-1`, `automat-2`, `miniload`, `truck-area`, `baler`, `battery-room`, `goods-in`, `dispatch`, `gate-1…gate-3`, `ramp`, `other-loc`

## 4. Fremtidige integrationsmuligheder

Datamodellen er bevidst ikke bundet til én backend. Den kan senere forbindes via fx:

- **REST API**: `POST` af `exportReportAsJSON()` til kundens endpoint
- **Webhook**: push ved oprettelse/statusskift
- **ERP / WMS / QMS**: mapping af `object_id`, `qr_code`, `department_id`, `supplier_id`
- **Microsoft-systemer** (fx Power Automate, SharePoint-lister, Dataverse)
- **SAP** (fx via middleware / IDoc / OData)
- **Kundens egen database**

Implementering sker i `sendToExternalSystem()`, når Clip-Loks IT har valgt løsning. Samtidig flyttes billeder til ekstern storage, og `storage_reference`, `external_reference` og `integration_status` udfyldes.

## 5. QR-klargøring

`object_id` og `qr_code` findes i datamodellen. Der er ingen QR-scanning eller QR-administration endnu. Felterne gør det senere muligt at koble rapporthistorik til konkrete transportkasser, racks, maskiner, reoler, trucks, porte og andre objekter.
