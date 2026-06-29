# Smartin API — full endpoint catalog

Verified working endpoints on dev (`workspace-service-235716971841.europe-west1.run.app`) as of 23.5.2026.

## Workspaces & Users

| Method | Path | Use | Notes |
|---|---|---|---|
| GET | `/workspace?size=500` | Admin list all workspaces | founder permissions; returns id, email, name, ownerName, tokens (credit balance) |
| GET | `/workspace/user/all` | List workspaces the current user can see | minimal: id + name only |
| GET | `/workspace/user/{keycloakUserId}` | Current user's workspace (token-scoped) | returns id, email, name, tokens |
| GET | `/users/me` | Keycloak user profile | keycloakId, name, phone, email, smartinFields, seniority |

## Case Config

| Method | Path | Use |
|---|---|---|
| GET | `/case-config/all` | List (id, caseKey, smartinField, name) — 4 fields only |
| GET | `/case-config/{id}` | Full detail (~20 fields) |
| POST | `/case-config` | Create OR update (id in body = update) |
| ~~PUT~~ | `~~/case-config/{id}~~` | **405 Method Not Allowed** |
| ~~PATCH~~ | `~~/case-config/{id}~~` | **405** |
| ~~GET~~ | `~~/case-types~~` | **404** — wrong endpoint name |

Full case_config object fields: `caseKey, smartinField, sourceTypes, baseSeverity, name, situationDescription, detectionDescription, calculationDescription, impactDescription, recommendation, sourceQuery, fallback, promptDescription, hypothesisTemplateA/B/C, researchField (object), researchFieldId (POST-only)`.

## Hypothesis Templates

| Method | Path | Use |
|---|---|---|
| GET | `/hypothesis/templates/smartinField/{FIELD}` | List (id + internalCode) |
| GET | `/hypothesis/templates/{id}` | Full detail (incl. costCredits, recommendedAssets) |
| POST | `/hypothesis/templates` | Create (flat body since 17.5.2026) |
| PUT | `/hypothesis/templates/{id}` | Update (full object in body) |

Template detail fields:
```json
{
  "id": "uuid",
  "internalCode": "pipeline_stalled B)",
  "name": "Pipeline stojí — standardní ověření",
  "description": "...stav reality + bullety + outcome věta...",
  "smartinField": "SALES",
  "templateDescription": "VARIANTA B — standardní...\n\nKONTEXT:\n...\n\nINSTRUKCE PRO AI AGENTA:\n...",
  "costCredits": 50,
  "recommendedAssets": ["Test Plan", "..."],
  "checklistFields": []
}
```

**Naming convention:** internalCode = `{case_type_key} {A|B|C})` — technical, stable, do not modify. name = lidský UI název per Smartin lexikon.

## Idea Templates (Kompas)

| Method | Path | Use |
|---|---|---|
| GET | `/idea-templates` | List 6 kostiček (CUSTOMER, PROBLEM, PRODUCT, SALES, PRICING, REVENUE) × A/B/C = 18 šablon |
| POST | `/idea-templates` | Create (wrapper body: `{ hypothesisTemplateRequest: {...} }`) |
| PUT | `/idea-templates/{id}` | Update (wrapper body) |

Kompas pattern is different from hypothesis/templates — keeps wrapper. Don't merge.

## Research Fields & Maturity Framework

| Method | Path | Use |
|---|---|---|
| GET | `/researchFields?page=0&size=1000&sort=name,asc` | Global pool — 483 items na dev |
| GET | `/researchFields/answers/workspace/{ws}` | Per-workspace odpovědi na maturity otázky |
| GET | `/maturity-framework/items/smartinField/{FIELD}` | Items per oblast (legacy view) |
| GET | `/maturity-framework/items/{id}` | Detail item |
| POST | `/maturity-framework/items` | Create or update (full object) |
| DELETE | `/maturity-framework/items/{id}` | Delete — check refs first (case_config could link) |

`fieldType` values: `CHECKLIST` (co splnit), `MINIMAL_ASSET` (důkaz). MVA links to parent CHECKLIST via `checklistFields: [{ id, name }]`.

Levels: `ONE, TWO, THREE, FOUR, FIVE`. Audit covers L1–L3 = 20+20+20 = 60 položek per oblast plus 10+10 = 20 L4/L5 (edukační).

## Case Results & Detection

| Method | Path | Use |
|---|---|---|
| GET | `/case-result/overview/{ws}` | Overall status per oblast (RISK / DECISION_NEEDED / STUCK / OK / UNCLEAR) |
| GET | `/case-result/overview/{ws}/smartinField?smartinField={FIELD}` | Detail oblasti + signály + nextStepProposal |
| PUT | `/case-evaluation/case-config?workspaceId={ws}&caseConfigId={cc}` | Spustit "Vyhodnotit" pro konkrétní case |

Response shape pro overview/smartinField:

```json
{
  "smartinField": "MARKETING",
  "resultGroup": "RISK",
  "lastUpdated": "2026-05-19T15:18:12Z",
  "aggregatedResults": {
    "DECISION_NEEDED": [...],
    "OK": [...],
    "RISK": [...],
    "STUCK": [...],
    "UNCLEAR": [...]
  },
  "signalTexts": ["Měření nefunguje nebo chybí", "Konverze klesají", "..."],
  "nextStepProposal": "..."
}
```

## Hypotheses (workspace instances)

| Method | Path | Use |
|---|---|---|
| GET | `/hypothesis/workspace/{ws}?size=200` | Paginated list všech hypotéz v workspace |
| GET | `/hypothesis/{id}` | Detail jedné hypotézy |

Statuses: `DRAFT`, `TESTING`, `SUCCESSFUL`, `UNSUCCESSFUL`.

## Auth pattern

Always include `Authorization: Bearer {token}` header. Token from React fiber (see SKILL.md token extraction). Token from dev does not work on prod and vice versa.

## Common errors

| Status | Cause | Fix |
|---|---|---|
| 401 | Token expired or wrong env | Re-extract from current page |
| 400 | Field mismatch (doc names vs API) | Map per "Field mapping" tables |
| 404 | Wrong path or non-existent resource | Check exact endpoint pattern |
| 405 | Wrong method (PUT/PATCH instead of POST) | Use POST with full body |
| 500 | Server-side, often during sync gaps | Retry after 30s; if persists, check API status |
