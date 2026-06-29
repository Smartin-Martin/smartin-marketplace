---
name: smartin-api-access
description: Smartin API recipes and authentication. Use this skill whenever the user needs to read or write data in the Smartin app — case configs, hypothesis templates, research fields, case results, workspaces, maturity answers. Triggers on "Smartin API", "case-config", "hypothesis/templates", "researchFields", "case-result", "/workspace/", "dev.mjobs.blog", "smartin.work", "matula", "token", "401", "Vyhodnotit", "Aktualizovat". Loads recipes for token extraction from browser, common GET/POST patterns, and known field mappings.
---

# Smartin API Access

Two environments, identical schemas, different data. Always declare which one you're working on before any non-GET call.

## 1. Environments

| Parameter | DEV | PROD |
|---|---|---|
| Frontend | https://dev.mjobs.blog | https://smartin.work/chat |
| API base | `https://workspace-service-235716971841.europe-west1.run.app` | `https://api.smartin.work` |
| Keycloak realm | dev | `https://iam.smartin.work/realms/smartin` |
| Martin's workspace ID | `8af70d48-70f9-4b41-9945-48c65aa29657` (SMARTIN COMPANY) | `8af70d48-70f9-4b41-9945-48c65aa29657` (stejné — sync zachoval UUIDs) |

**Důležité o workspace ID:** Workspace UUIDs jsou **sdílené mezi dev a prod**. dev→prod sync (poprvé 19.5.2026) zachoval všechna existující workspace ID. Tedy stejný klient má stejný workspace ID v obou prostředích.

`5e76b09d-040a-4642-a690-6d8373e4c9b9` je **Martinův Keycloak userId** (sub claim v JWT), ne workspace ID. (Memory pre-25.5.2026 toto zaměňuje.)

**Rule.** Default = dev. Switch to prod only on explicit user confirmation. Tokens are not interchangeable across environments.

## 2. Token extraction (browser, React fiber)

The token is not in cookies or localStorage. It lives in React component state. Extract via fiber tree traversal:

```javascript
function getTokenFromFiber() {
  const root = document.getElementById('root');
  const containerKey = Object.keys(root).find(k => k.startsWith('__reactContainer'));
  const container = root[containerKey];
  function find(fiber, depth = 0, visited = new Set()) {
    if (!fiber || depth > 60 || visited.has(fiber)) return null;
    visited.add(fiber);
    if (fiber.memoizedProps && typeof fiber.memoizedProps === 'object') {
      for (const [key, val] of Object.entries(fiber.memoizedProps)) {
        if (val && typeof val === 'object' && val.token && typeof val.token === 'string' && val.token.length > 100) {
          return 'Bearer ' + val.token;
        }
      }
    }
    return find(fiber.child, depth + 1, visited) || find(fiber.sibling, depth + 1, visited);
  }
  return find(container);
}
window._token = getTokenFromFiber();
```

**Re-extract whenever API calls return 401.** SPA navigation rebuilds fiber tree.

## 3. Core endpoint patterns

Plný katalog je v `references/endpoints.md`. Tady jsou nejčastější.

### Workspaces

```
GET /workspace/user/all                          // list of all workspaces visible to current user
GET /workspace/user/{keycloakUserId}             // info about current user's workspace (token-scoped)
GET /workspace?size=500                          // admin list (founder permissions)
GET /users/me                                    // current Keycloak user details (smartinFields, seniority)
```

`activeWorkspaceId` is stored in localStorage key `mjobs.activeWorkspaceId:{keycloakUserId}`. To impersonate another workspace as founder, set this value and reload — backend honors it for users with workspace claim.

### Case Config (typy případů)

```
GET  /case-config/all                            // list (id, caseKey, smartinField, name)
GET  /case-config/{id}                           // detail
POST /case-config                                // create OR update (with id in body)
```

**Trap.** PUT and PATCH return 405. Update = POST with full object.

**Field mapping doc → API:**

| Doc název | API pole |
|---|---|
| title_human | name |
| what_happens | situationDescription |
| why_it_matters | impactDescription |
| nba | recommendation |

`titleHuman`, `whatHappens`, `whyItMatters`, `nba` v API NEEXISTUJÍ a vrátí 400.

### Hypothesis Templates

```
GET  /hypothesis/templates/smartinField/{FIELD}  // list (id + internalCode only)
GET  /hypothesis/templates/{id}                  // full detail
POST /hypothesis/templates                       // create (flat body since 17.5.2026, no wrapper)
PUT  /hypothesis/templates/{id}                  // update (different pattern than case-config)
```

`smartinField` values: `MARKETING`, `SALES`, `CUSTOMER_SUCCESS`, `OPERATIONS`, `CUSTOMER_MARKET`, `PRODUCT`.

Internal code pattern: `{case_type_key} {A|B|C})`. Prices: 10/50/150 credits.

### Idea Templates (Kompas kostičky)

```
POST /idea-templates                             // wrapper body: { hypothesisTemplateRequest: {...} }
PUT  /idea-templates/{id}                        // wrapper body
```

**Kompas POST/PUT keep the wrapper** even after the hypothesis/templates change. Don't merge these patterns.

### Research Fields & Maturity

```
GET  /researchFields?page=0&size=1000&sort=name,asc            // global pool of items
GET  /researchFields/answers/workspace/{workspaceId}            // per-workspace answers (CHECKLIST + MVA)
POST /maturity-framework/items                                  // create or update item (full object)
DELETE /maturity-framework/items/{id}                           // delete (check refs first)
```

`fieldType`: `CHECKLIST` (co splnit) + `MINIMAL_ASSET` (důkaz). MVA name pattern: `{prefix} asset: {evidence}`. MVA links via `checklistFields: [{ id, name }]` to parent CHECKLIST.

### Case Results & Detection

```
GET  /case-result/overview/{workspaceId}                                    // all 6 areas overall status
GET  /case-result/overview/{workspaceId}/smartinField?smartinField={FIELD}  // single area detail with signals
PUT  /case-evaluation/case-config?workspaceId={ws}&caseConfigId={cc}        // trigger "Vyhodnotit"
```

**Vyhodnotit, ne Aktualizovat.** UI tlačítko "Aktualizovat" je broken. Vždy přes edit form a tlačítko "Vyhodnotit". API endpoint výše je správný backend call.

### Hypotheses (instance, ne templates)

```
GET /hypothesis/workspace/{workspaceId}?size=200    // all hypotheses in workspace
```

Returns paginated list with: `id, hypothesisTitle, hypothesisText, target, status, smartinField, createdAt, updatedAt`.

Statuses: `DRAFT`, `TESTING`, `SUCCESSFUL`, `UNSUCCESSFUL`.

### Hypothesis creation (UI flow, no direct API)

Backend nemá `POST /hypothesis` endpoint pro creation from template. Vytvoření hypothesis = UI flow přes `/chat/case-result`.

UI path:
1. Navigate `/chat/overview/{FIELD}` (např. `/chat/overview/SALES`)
2. Click target case button (např. "Málo příležitostí v pipeline") → lands na `/chat/case-result/{caseResultId}`
3. Click "Ověřit" button → lands na `/chat/hypothesis/new` se 3 variant cards
4. Click variant card (`Pipeline neroste — rychlé/standardní/hloubkové ověření`) — MuiCard-root parent
5. Click "Pokračovat k ověření" button → confirmation step
6. Click "Zaplatit {10|50|150} kreditů a vytvořit ověření" → backend LLM call (~15–20 s)
7. Page redirects to `/chat/hypothesis/{id}` — DRAFT status, paid, not yet validated

Pro automatizaci přes Claude in Chrome: 4 clicks + 1 wait per hypothesis. Token re-extract typicky potřeba mezi navigations (SPA rebuilds fiber).

## 4. Common headers

```javascript
const headers = {
  'Authorization': window._token,  // includes "Bearer " prefix
  'Content-Type': 'application/json'
};
```

## 5. Known traps

- **Detection is non-deterministic** for SALES. LLM-based detection returns varying severity each Vyhodnotit, even without data change. Don't expect deterministic results when testing HubSpot changes.
- **researchField asymmetry.** GET returns `researchField: { id, name }` nested object. POST expects `researchFieldId: "<uuid>"` string. When updating case_config, `delete researchField; set researchFieldId`.
- **sourceTypes enum.** Use `HUBSPOT` and `GOOGLE_ANALYTICS`. Value `GA4` returns 400.
- **CUSTOMER_SUCCESS enum** was added automatically on first POST in 14.5.2026. Localizes in UI as "Péče o zákazníka".

## 6. Output limit in browser javascript_tool

`javascript_tool` truncates output around 1.5KB. For large data, save into `window._x` and return summary only. Hash-based diff for comparing across instances.

**Filter trap.** `javascript_tool` výstup obsahující stringy `token`, `auth`, `bearer` (case-insensitive jako keys nebo values) je blokovaný a vrací `[BLOCKED: Sensitive key]`. Workaround: použít neutrální názvy proměnných v return statementu (`{ok: true, len: ...}` místo `{hasToken: true, tokenLength: ...}`). `window._token` jako úložná proměnná zůstává OK — blokuje se jen output filter.

## 7. References

`references/endpoints.md` — full endpoint catalog with sample payloads.

## 7. HubSpot jako úložiště klientské konfigurace

Plugin čte profil klienta z HubSpot company `description` (volitelně z custom property `smartin_config`). Formát = free-form YAML. Validováno 25.5.2026 na SMARTIN company (HubSpot id 423340960965).

### Čtení config

```javascript
// Through HubSpot MCP tool — get_crm_objects with company id
// Returns properties.description as plain text (YAML stored as string).
// Plugin parses YAML in-memory; no schema enforcement, LLM extracts intent.
```

Klíčové pole:

| YAML klíč | Co řídí | Příklad |
|---|---|---|
| `team.daily_owners.{area}` | Phase 4 alokační tabulka pre-populated | `sales: Martin` |
| `active_sales_team_members` | sourceQuery overlay pro Sales detekci | `[Veronika, Tom, Petra]` |
| `priorities_this_quarter` | Phase 3 read-out kontext, weekly review framing | seznam vět |
| `dont_touch_now` | Phase 4 skip — areas, které se neallokují | seznam oblastí |
| `tone` | Decision memo + chat odpovědi style | `bias to action`, `cs preferred` |

### Zápis config

Plugin zapisuje při Phase 0.5 (po dialogu s klientem) nebo kdykoliv klient řekne „přidej do obchodníků X" / „změň priority na Y":

```javascript
// Through HubSpot MCP — manage_crm_objects with updateRequest
// Field = description; value = YAML string
```

### Failure modes pro HubSpot config

- Description prázdná → plugin se zeptá v Phase 0.5.
- Description nevypadá jako YAML → plugin parsuje jako prose a extrahuje fakta LLM cestou.
- Klient nemá HubSpot → použij note na Slack channel nebo lokální YAML soubor v Drive klienta.
- Custom property `smartin_config` (pokud existuje na company objektu) má přednost před `description`.

## 8. HubSpot Notes jako Decision Memory storage

Decision memos žijí v HubSpot jako Notes asociované s company recordem klienta. Validováno 25.5.2026 na SMARTIN company.

### Vytvořit nový decision memo

```javascript
// Through HubSpot MCP — manage_crm_objects createRequest
// objectType: "notes"
// properties.hs_note_body = markdown text
// properties.hs_timestamp = ISO 8601
// associations = [{ targetObjectType: "companies", targetObjectId: <company id> }]
```

Body podporuje markdown (bold, lists, hashtags). Tagy přes `#onboarding`, `#weekly-review`, `#exception`, `#phase-{n}` pomáhají filtraci v HubSpotu.

### Číst historii memos

```javascript
// search_crm_objects, objectType: "notes"
// filterGroup.associatedWith = [{ objectType: "companies", operator: "EQUAL", objectIdValues: [<company id>] }]
// properties: ["hs_note_body", "hs_timestamp", "hs_createdate"]
// sorts: [{ propertyName: "hs_timestamp", direction: "DESCENDING" }]
```

Vrátí všechny memos pro daný klient, řazené od nejnovějších. Plugin v daily / weekly review tahá top N a používá je jako kontext.

### Pattern pro plugin

| Trigger | Akce | Tagy |
|---|---|---|
| Phase 3 read-out hotový | create note se shrnutím + exceptions | `#phase-3` `#read-out` |
| Phase 5 hypotéza otevřena | create note: kdo otevřel, co, status DRAFT, link | `#phase-5` `#hypothesis-draft` |
| Týdně v pátek | create note: weekly review (Reality / Interpretace / Next step) | `#weekly-review` |
| Klient řekne „překvapilo nás" | create note: exception log | `#exception` |
| Klient žádá feature | create note: feature request | `#product-signal` |

### Dvě úložiště, dvě role

- `description` property na company = **statický config** (`smartin_config` YAML). Mění se zřídka, drží stabilní fakta o firmě.
- HubSpot Notes na company = **dynamický rolling log** (decision memory). Append-only, časem narůstá, klient vidí historii na timeline.

Nemíchat. Plugin čte config z `description` při startu session, ale memory ze Notes čte při daily/weekly briefu.

## 9. Čtyři sdílená úložiště — kde se co synchronizuje

Plugin v každém uživatelově Coworku má vlastní chat session. Tento chat **není sdílený**. Synchronizace mezi uživateli běží přes čtyři sdílená úložiště, do kterých plugin čte a píše.

| Úložiště | Co tam je | Read frequency | Write trigger |
|---|---|---|---|
| Smartin backend (api.smartin.work) | Mapa firmy, hypotézy, case results, maturity odpovědi, workspace data | při startu session + při explicit refresh | po každé akci v Phase 2/5/6 (přes API call) |
| HubSpot `description` na company | smartin_config YAML (statický profil firmy) | při startu session (Phase 0.5) | při změně config (Phase 0.5 update, Phase 4 owner change) |
| HubSpot Notes na company | rolling log decision memos (dynamický) | při daily brief + weekly review | po každém decision (Phase 3 read-out, Phase 7 weekly memo, exceptions) |
| Slack `#klient-{slack_channel_name}` | live notifikace + lidská diskuze | průběžně (přes Slack MCP) | po každé akci (`[ACTION]`, `[MEMO]`, `[NUDGE]`) |

Drive `09_Klienti/{klient}/decision_memos/` slouží jako long-term archive — plugin tam jednou týdně synchronizuje weekly memo jako markdown soubor.

**Co se NEsynchronizuje:**

- Chat history v individuálním Coworku — to je osobní pracovní prostor uživatele, nikdo jiný ho nevidí.
- Brainstorming bez akce — když owner ve svém Coworku přemýšlí „mám otevřít X nebo Y", ostatní to nevidí, dokud plugin nespustí konkrétní akci.

**Důsledek pro architekturu.** Synchronizace neběží mezi Coworky. Běží mezi Coworky a sdílenými úložišti. Každý uživatel se „dívá" do stejného obrazu skrze svůj plugin.

## 10. Per-klient overlay přes HubSpot

Generická Smartin detekce počítá s default populací (např. „všichni HubSpot useři jsou potenciální obchodníci"). Pro konkrétní klient to často není pravda. Klient má 10 userů v HubSpotu, ale jen 3 z nich opravdu obchodují.

**Overlay storage = HubSpot List.**

Klient vytvoří v HubSpotu List s názvem např. „Smartin: Active Salespeople". Přidá tam 3 user records, kteří jsou skuteční obchodníci. Plugin čte members tohoto Listu a filtruje sourceQuery detekce.

```javascript
// search_crm_objects, objectType: "users" (or "owners")
// Filter via list membership using HubSpot list API
// Returns 3 active salesperson IDs for Legal Systems
```

Pattern Lists per oblast:

| Oblast | List name | Co tam patří |
|---|---|---|
| Sales | „Smartin: Active Salespeople" | useři, kteří aktivně obchodují |
| CSM | „Smartin: CSM Team" | useři, kteří mají kontakt s klienty |
| Operations | „Smartin: Ops Team" | useři, kteří řídí provoz |
| Marketing | „Smartin: Tracked Campaigns" | kampaně, které sledujeme (campaign objects, ne users) |

**Alternativa — custom property na user objektu.** Pokud klient nechce udržovat Lists, lze přidat custom property `smartin_role` na User object s enum hodnotami `salesperson / csm / ops / marketing / admin / none`. Plugin filtruje po této property.

**Setup v onboardingu.** V Phase 1 (Connect data) plugin po napojení HubSpotu projde HubSpot users a zeptá se klienta „kdo z těchto userů je obchodník? Kdo CSM?" Po odpovědi vytvoří Lists nebo nastaví property. Klient si pak může v HubSpotu manuálně updatovat.

**Důležité.** Plugin Listy nevytváří automaticky bez potvrzení klienta. Akce „create HubSpot List" vždy projde explicitním souhlasem v dialogu.

## 11. Slack hranice pluginu vs. konzultanta

Plugin je obousměrně napojený na Slack přes Cowork MCP (čte i píše). Ale ne každá obousměrnost je dobrá. Hranice je striktní.

### Co plugin DĚLÁ

| Akce | Komu |
|---|---|
| Píše decision memos do `#klient-{slack_channel_name}` po každé větší akci (`[MEMO]`) | klientovi |
| Píše action notifikace (`[ACTION]` typu „Anna otevřela hypotézu") | klientovi |
| Píše nudge připomínky (`[NUDGE]`) v konzultant mode | klientovi přes konzultanta |
| Čte historii `#klient-{slack_channel_name}` v konzultant mode | pro konzultanta |
| Klasifikuje klientovy zprávy a vytváří konzultant queue | pro konzultanta |
| Generuje denní digest pro konzultanta („co dnes musíš odpovědět") | konzultantovi v jeho Coworku |

### Co plugin NEDĚLÁ

| Akce | Důvod |
|---|---|
| Neodpovídá automaticky na klientovy zprávy v Slacku | Slack je lidský kanál mezi konzultantem a klientem. AI auto-reply ničí důvěru. |
| Nepřidává emoji reakce na zprávy klienta | Působí strojově. |
| Neeskaluje proaktivně do jiných Slack kanálů | Konzultant rozhoduje, kam co posunout. |
| Neagreguje obsah napříč klientskými kanály v self-serve/konzultant mode | Cross-tenant leak rizik. Pouze internal mode má aggregator. |

### Klientova strana

Klient ve svém Coworku se může pluginu explicitně zeptat „kolega minulý týden něco psal o pipeline, najdi to". Plugin pak vyhledá v `#klient-{slack_channel_name}` (klient má k tomu kanálu přístup) a vrátí kontext. Klient iniciuje, plugin nečte proaktivně cizí zprávy.

### Konzultantova strana

Konzultant (Smartin tým) v ranním briefu vidí:

> V noci v #klient-ls napsal Tom dvě zprávy:
> 1. [BUG] problém s napojením HubSpotu na Sales detekci — viz konverzace 23:45
> 2. dotaz na maturity audit — viz konverzace 00:12
>
> První je urgentní (failure handler 2 — HubSpot napojení). Druhé počká do weekly review.

Konzultant pak odpovídá osobně. Plugin pracuje pro konzultanta, ne za něj.

## 12. Prod validace 25.5.2026

API endpointy ověřeny na `api.smartin.work` v rámci přípravy LS onboarding:

| Endpoint | Status | Hodnota |
|---|---|---|
| `/workspace?size=500` | 200 | 13 workspaců |
| `/case-config/all` | 200 | **60 case_configs** (10 × 6 oblastí) |
| `/hypothesis/templates/smartinField/SALES` | 200 | 37 šablon |
| `/hypothesis/templates/smartinField/MARKETING` | 200 | 37 šablon |
| `/researchFields?size=1000` | 200 | 481 fields (parita s dev 483) |
| `/users/me` | 200 | Token-scope OK |
| `/case-result/overview/{ws}` | 200 | Mapa firmy data |

**LS workspace na prod:** `bf8bf432-75b5-43c8-86b8-56a58e93d749` (stejné ID jako dev). Aktivní stav k 25.5.2026:
- MARKETING: RISK (už vyhodnoceno)
- SALES: RISK (už vyhodnoceno)
- CSM / OPS / C&M / PRODUCT: čekají na první Aktualizaci v pondělí

Token extraction z `smartin.work/chat` fiber tree funguje identicky jako z `dev.mjobs.blog`.

## 12.5. Google Drive — primární úložiště Decision Memory

**Důležitá změna od v0.6:** Decision Memory, config, session state, friction logy, Kompas deliverables — **vše leží v Smartin gdrive**, ne v HubSpot Notes. HubSpot Notes je opcionální konzultantův backup (default vypnuto). Důvod: klient nemá přístup do Smartin HubSpotu, ale má přístup do sdíleného Drive folderu. Plus chceme stavět bez backend dev týmu, dokud Smartin app feature nepřijde.

Kanonický popis struktury: `04_Produkt/smartin-beta-plugin/struktura_09_klienti.md`.

**Drive recipes (Google Drive MCP):**

```javascript
// Vytvořit folder
await drive.create_file({
  name: "Legal Systems",
  parents: ["{id_of_09_Klienti}"],
  mimeType: "application/vnd.google-apps.folder"
});

// Vytvořit pod-foldery (decision_memos, weekly_drafts, friction_logs, kompas, exports)
// Stejně pro každý

// Zapsat smartin_config.yaml
await drive.create_file({
  name: "smartin_config.yaml",
  parents: ["{id_of_klient_folder}"],
  mimeType: "text/yaml",
  content: yamlString
});

// Číst session_state.yaml
const content = await drive.read_file_content({fileId: "{id}"});
const state = parseYaml(content);

// Ověřit, kdo má přístup
const perms = await drive.get_file_permissions({fileId: "{folder_id}"});
const sharedEmails = perms.permissions.map(p => p.emailAddress);
// porovnat s team.daily_owners
```

**Sdílení folderu — plugin NEDĚLÁ.** Security restriction: plugin nemůže měnit permissions. Místo toho plugin **instruuje konzultanta**, aby folder nasdílel ručně v Drive UI. Po klientském potvrzení plugin přes `get_file_permissions` ověří, že emaily v `team.daily_owners` mají přístup.

**HubSpot Notes deprecation.** API recipes z minulých verzí (`create_note`, `add_note_to_company`) zůstávají v failure handlers a konzultantský backup workflow. Pro Decision Memory primárně **gdrive**, ne HubSpot. Pokud konzultant chce HubSpot backup, plugin to volá paralelně po commitu memo.

## 13. Draft-first pravidlo pro všechna client-facing communication channels

**SYSTÉMOVÉ PRAVIDLO (od v0.9).** Plugin NIKDY nepošle zprávu klientovi
bez explicitního konzultantova „ano". Platí pro:

- Slack Connect channels (externí workspaces)
- Slack public channels (interní Smartin)
- Slack DM klientovi
- Email (Gmail MCP — create_draft, ne send_message)
- Jakýkoliv future kanál (Teams, WhatsApp, atd.)

Plugin vždy:
1. Vytvoří draft (Slack draft, Gmail draft) — viditelný konzultantovi.
2. Pošle Slack notifikaci konzultantovi „[DRAFT] připraveno k review: {link}".
3. Konzultant ručně reviduje a odešle.

Bez výjimky. Důvod: LS handoff 26.5.2026 + Meta IT learnings 27.5.2026 — plugin v ostrých kanálech psal elaboráty, konzultant musel mazat.

`#klient-{slack_channel_name}` je z principu Slack Connect (Smartin workspace × klientův workspace). Slack na úrovni platformy omezuje, co může app instalovaná v jednom workspace v Connect kanálu dělat.

**Co funguje v Slack Connect:** `slack_send_message_draft`, `slack_read_channel`, `slack_read_thread`.

**Co failuje:** `slack_send_message`, `slack_add_reaction` (vrací `missing_scope` / `channel_not_found` / `not_in_channel`).

**Detection (před prvním zápisem):**

```javascript
const result = await slack_search_channels({
  query: "klient-ls",
  channel_types: "public_channel,private_channel"
});
const isConnect = result.channels[0].is_shared === true;
```

**Pokud isConnect, plugin přepíná na draft mode pro VŠECHNY posty (preventivně):**

```javascript
await slack_send_message_draft({ channel: channel.id, text: memo });
// notifikace konzultantovi
await slack_send_message({
  channel: "#smartin-ops",
  text: `[DRAFT READY] #klient-ls — weekly memo čeká na odeslání`
});
```

Konzultant ráno (~5 min/den) projde drafts a odešle ručně. Plugin sám neodesílá za konzultanta. Onboarding-guide Phase 0.5 to klientovi i konzultantovi řekne dopředu.

Failure handler `#13 Slack Connect` v `smartin-failure-handlers` skill.

## 14. Konzultant portfolio query

V konzultant módě plugin při startu session zavolá portfolio recipe — jedno volání místo 3 separátních.

```javascript
// 1. Companies, které jsou Smartin klienti
const companies = await search_crm_objects({
  objectType: "companies",
  filterGroups: [{
    filters: [{
      propertyName: "description",
      operator: "CONTAINS_TOKEN",
      value: "smartin_workspace_id"
    }]
  }],
  properties: ["name", "domain", "description"],
  limit: 50
});

// 2. Pro každého parse YAML z description
const portfolio = [];
for (const c of companies.results) {
  const config = parseYaml(c.properties.description);
  // 3. /case-result/overview/{ws}
  const overview = await fetch(`${API_BASE}/case-result/overview/${config.smartin_workspace_id}`);
  portfolio.push({
    name: c.properties.name,
    sponsor: config.team.sponsor,
    phase: config.session_state.current_phase,
    days_in_phase: daysBetween(config.session_state.last_action_at, now),
    blocked_on: config.session_state.blocked_on,
    areas: overview.areas
  });
}

// 4. Sort: blocked first, stagnation (> 4 dny) next, recency last
portfolio.sort(byPriority);
```

`STAGNATION_THRESHOLD` = 4 dny per Phase (heuristika, tunit per klienta v YAML).

**Ranní brief v konzultant módě:**

```
Ráno k 09:00, tvoje portfolio:

⚠️ Legal Systems — blocked: čeká na Tom HubSpot napojení (3 dny)
🟡 SMARTIN COMPANY — Phase 3, 6 dní v fázi (stagnation)
🟢 Petotest — Phase 0.5 (sandbox)
🟢 Frank Bold — Phase 7, weekly review v pátek

Komu věnovat čas dnes? [Legal Systems / SMARTIN / něco jiného]
```

## 15. Slack search — defaults a retry

**Search defaults.** Vždy přidat private channels:

```javascript
slack_search_channels({
  query: "klient-ls",
  channel_types: "public_channel,private_channel"
});
```

Anti-pattern: `slack_search_channels({query: "klient-ls"})` — default je `channel_types="public_channel"`. Klientské kanály jsou často private, default vrátí 0 a plugin si myslí, že kanál neexistuje.

**Retry logika.** Pokud první search vrátí 0 výsledků s explicit channel_types, plugin re-tryne bez filtru jako fallback.

## 16. ResearchField answers + case-evaluation trigger (Phase 0.7 + 6)

Plugin v Phase 0.7 (9 outside-in checklistů) a Phase 6 (full maturity audit) zapisuje klientovy odpovědi přes RF API a pak trigne re-evaluation.

**Zapsat odpověď na CHECKLIST:**

```javascript
await fetch(`${API_BASE}/researchFields/workspace/${ws}/answer/detail/${rfId}`, {
  method: "PUT",
  headers: { Authorization: `Bearer ${token}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    answer: "DONE",          // DONE | NOT_DONE | PENDING | SKIPPED
    answerOption: "DONE"     // POVINNÉ! UI dropdown čte z answerOption, ne z answer
  })
});
```

**KRITICKÁ PAST (objevena 7.6.2026 při LS handoff):**

RF answer má **dvě pole**, obě se musí nastavit shodně:

- `answer` (text label) — backend detection ho čte, dělá detection na semafory
- `answerOption` (enum) — UI dropdown ho čte, ukazuje badge "Hotovo / Nezodpovězeno / Čeká / Přeskočeno"

Pokud zapíšeš jen `answer` a vynecháš `answerOption`, **backend detection funguje, ale UI dropdown zůstane prázdný**. Klient otevře RF detail, vidí v textovém poli "DONE" ale dropdown bez badge → ztratí důvěru ve výsledek.

Mapování dropdown ↔ enum (empiricky potvrzeno):

| UI label | answer + answerOption |
|---|---|
| Hotovo | `DONE` |
| Nezodpovězeno | `NOT_DONE` |
| Čeká | `PENDING` |
| Přeskočeno | `SKIPPED` |

**Pravidlo:** vždy nastavit `answer = answerOption`. Bez výjimky.

**Nahrát MVA důkaz (volitelně po DONE odpovědi):**

Plugin nemůže nahrát file přímo přes Smartin API (není dedikovaný upload endpoint pro RF assets). Workaround:

1. Plugin uloží file do `09_Klienti/Smartin × {klient}/exports/MVA/{oblast}/{rf_id}.{ext}` přes Drive MCP `create_file`
2. Plugin do `smartin_config.yaml` zapíše link: `mva_assets: {rf_id: drive_file_id}`
3. Plugin do `decision_memos/{date}_phase_0_7_mva.md` zapíše seznam uploaded assets pro auditní trail

**Trigger re-evaluation per oblast:**

```javascript
await fetch(`${API_BASE}/case-evaluation/workspace?workspaceId=${ws}&smartinField=MARKETING`, {
  method: "PUT",
  headers: { Authorization: `Bearer ${token}` }
});
```

**Pozor:** Backend padá pod concurrency > 1 (viz LS handoff F3). Plugin volá sekvenčně, ne paralelně. Po každém PUT pollne `/case-result/overview/{ws}`:

```javascript
async function pollUntilFresh(ws, areas, maxSeconds = 60) {
  const start = Date.now();
  while (Date.now() - start < maxSeconds * 1000) {
    const overview = await fetch(`${API_BASE}/case-result/overview/${ws}`);
    const data = await overview.json();
    if (areas.every(a => isFresh(data.caseResultOverview[a]))) return data;
    await sleep(5000);
  }
  return null;  // timeout — plugin to handle gracefully
}
```

Pokud poll timeoutuje, plugin neblokuje session — řekne klientovi „Aktualizace běží na pozadí, podívám se na to a pošlu ti Slack notifikaci."

**Pre-check pro UNAVAILABLE oblasti (Phase 3 fix F2):**

Před re-evaluation plugin ověří, jestli oblast má napojené data. Pokud ne, vrátí oblast jako UNAVAILABLE bez volání case-evaluation (nezbytné — placeholder STUCK signaly halucinují, viz F2):

```javascript
const REQUIRED_CONNECTORS = {
  "MARKETING": ["GA4", "Tag Manager", "HubSpot", "Mailchimp"],
  "SALES": ["HubSpot", "Pipedrive", "Salesforce"],
  "CUSTOMER_SUCCESS": ["Intercom", "Crisp", "Help Scout", "Zendesk"],
  "OPERATIONS": ["Stripe", "FAPI", "calendar"],
  "CUSTOMER_MARKET": [],  // nemá specifický connector, vždy available
  "PRODUCT": ["GitHub", "Linear", "Jira"]
};

function isAreaAvailable(area, clientConnectors) {
  const required = REQUIRED_CONNECTORS[area];
  if (required.length === 0) return true;
  return required.some(c => clientConnectors.includes(c));
}
```

## 17. Smartin MCP server — konzultantský primary path

V plugin v0.8+ je `mcp__smartin__*` server **primary cesta** pro konzultantské write operace. Chrome navigation + token extraction zůstává jako fallback (klient v jeho Coworku, MCP server down).

**Kdy MCP, kdy Chrome:**

| Situace | Použít |
|---|---|
| Konzultant (Martin) v jeho Coworku | MCP primary |
| Klient v jeho Coworku | Chrome + token (MCP server klient nemá) |
| MCP server vrátí 5xx nebo timeout | Chrome fallback |
| Read RF answers, write RF answers, trigger reeval | MCP (od 5. 6. 2026 plně podporováno) |

**20 dostupných MCP toolů (po update 5. 6. 2026):**

Read operations:
- `smartin_list_workspaces` → 18 workspaces, is_superadmin flag
- `smartin_get_overview(workspace_id)` → semafory per 6 oblastí
- `smartin_get_overview_by_field(workspace_id, smartin_field)` → 1 oblast
- `smartin_list_case_configs(workspace_id)` → 60 case_configs
- `smartin_get_case_config(case_config_id, workspace_id)` → detail
- `smartin_list_research_fields(workspace_id, page, size)` → 476 RFs (jen definice)
- `smartin_get_research_field_answer(workspace_id, research_field_id)` → **NEW 5.6.** current answer + lastModifiedBy
- `smartin_list_templates`, `smartin_get_template` → hypothesis templates

Write operations (konzultant superadmin scope):
- `smartin_upsert_research_field_answer(workspace_id, research_field_id, answer, answer_option)` → **NEW 5.6.** write RF answer (Karel use case)
- `smartin_upsert_case_config`, `smartin_delete_case_config` → admin
- `smartin_create_research_field`, `smartin_delete_research_field` → admin
- `smartin_create_template`, `smartin_update_template` → admin

Trigger operations:
- `smartin_evaluate_case(case_config_id, workspace_id)` → reeval 1 case, async ~10–15 s
- `smartin_evaluate_batch(workspace_id, case_config_ids[])` → batch reeval
- `smartin_evaluate_workspace(workspace_id, smartin_field?)` → **NEW 5.6.** reeval celá oblast nebo workspace, fix concurrency 500 storm z LS handoffu F3

**Recipe: Karelův LS sales maturity backfill (po 5. 6. 2026):**

```python
ws = "bf8bf432-75b5-43c8-86b8-56a58e93d749"   # Legal Systems

# 1. Pre-check existing state (ať nepřepíšu Karelovy předchozí odpovědi)
for rf_id, expected in karel_mapping.items():
    current = smartin_get_research_field_answer(workspace_id=ws, research_field_id=rf_id)
    if current.answer and current.answer != "PENDING":
        # už vyplněné, skip nebo confirm
        log_conflict(rf_id, current, expected)

# 2. Batch write — answer_option JE POVINNÉ (jinak UI dropdown zůstane prázdný)
for rf_id, answer in karel_mapping.items():
    if answer != "PENDING":  # PENDING přeskakujeme, klient sám doplní v dialogu
        smartin_upsert_research_field_answer(
            workspace_id=ws,
            research_field_id=rf_id,
            answer=answer,
            answer_option=answer  # MUSÍ se shodovat s answer (objeveno 7.6.2026)
        )

# 3. Trigger reevaluation Sales oblasti
smartin_evaluate_workspace(workspace_id=ws, smartin_field="SALES")

# 4. Poll do fresh result (max 60 s)
overview = wait_for_fresh_overview(ws, areas=["SALES"], max_seconds=60)

# 5. Decision memo do Drive
write_decision_memo(klient="Legal Systems", date=today, content="""
Sales maturity backfill — 23 odpovědí zapsáno (13 DONE, 10 NOT_DONE).
3 PENDING odloženy do dalšího weekly review.
Sales oblast: STUCK → {overview.SALES} po reeval.
""")
```

**Bezpečnostní pravidla** (z `signaly/smartin_mcp_capability_27052026.md`):
- MCP **NIKDY klient**. Klient v jeho Coworku MCP server nemá.
- Write operations na produkci = vyžadují explicit „ano, na prod" od Martina (per CLAUDE.md)
- Před batch write: pre-check existing state přes `get_research_field_answer`, ať nepřepíšeš klientovy předchozí odpovědi (failure handler #17 governance)

**Failure fallback:** Pokud MCP volání vrátí 5xx nebo timeout, plugin spadne na Chrome + token extraction (sekce 1 + 16). Žádný retry kruh (methodology §10 pravidlo 11).

## 19. HubSpot client query (anti-pattern) — anti-pattern

```javascript
// Správný pattern — hledat marker v YAML configu
search_crm_objects({
  objectType: "companies",
  filterGroups: [{
    filters: [{
      propertyName: "description",
      operator: "CONTAINS_TOKEN",
      value: "smartin_workspace_id"
    }]
  }]
});
```

Marker `smartin_workspace_id:` v YAML configu zaručuje, že company je Smartin klient. Alternativa: custom HubSpot property `smartin_workspace_id` přímo na company objektu (cleaner, ale vyžaduje setup).

## 18. Email fallback path (Gmail MCP)

Pro klienty bez placeného Slacku (free Slack neumí Connect s externími workspaces), plugin běží na email + Drive. `communication_channel: email` v `smartin_config.yaml`.

Gmail MCP recipes:

1. **První email (vítej v Smartinu)** — po Phase 0.5 prep:
   - Volá `mcp__gmail__create_draft` (NEVOLAT send_message, draft-first rule)
   - Subject: „Smartin × {Klient} — start"
   - Body: 5–8 řádků (kdo jsem, co Smartin dělá, co bude D-day, Drive link)
   - Konzultant draft reviduje v Gmail UI a odešle ručně

2. **Decision memo notifikace** — když plugin uloží memo do Drive:
   - `create_draft` s subject „[Smartin memo] {date} weekly review"
   - Body: 3 řádky shrnutí + Drive link
   - Konzultant reviduje + odesílá

3. **Týdenní digest** — Phase 7:
   - `create_draft` s subject „[Smartin] týdenní digest W{N}"
   - Body: tabulka oblastí + 3 priority + Drive link
   - Konzultant reviduje + odesílá

Plugin NIKDY nepoužije `send_message` přímo — vždy `create_draft`. Konzultant kontroluje obsah před odesláním.

**Gmail label setup (jednou per klient):**

V Phase 0.5 prep plugin instruuje konzultanta:
1. V Gmailu vytvořit label `Smartin × {Klient}`
2. Filter: `from:{klient_email_domain}` → apply label
3. Plugin si label cachuje do `smartin_config.yaml` jako `gmail.label`

**Email channel + Slack channel paralelně možné** (`communication_channel: both`). Plugin pak posílá draft do obou — konzultant si vybere.

## 20. get_all_case_results recipe (pro snapshots skill)

Plugin při snapshot capture potřebuje všech 60 case-result detailů s plným textem + sourceQuery + metrics. Single GET endpoint pro batch neexistuje, takže paralelně.

```python
async def get_all_case_results(workspace_id):
    cases = await mcp__smartin__smartin_list_case_configs(workspace_id)
    case_ids = [c["id"] for c in cases]
    
    # Paralelně 60 calls (Smartin MCP supports concurrent reads)
    results = await asyncio.gather(*[
        get_case_result(workspace_id, case_id) for case_id in case_ids
    ])
    return results

def get_case_result(workspace_id, case_id):
    # Smartin MCP nebo Chrome+token fallback
    return mcp__smartin__smartin_get_case_result(
        workspace_id=workspace_id,
        case_config_id=case_id
    )
```

**Pozor:** některé case_results mohou být UNCLEAR (engine nedokáže rozhodnout). To je validní stav, ne error. Plugin zapíše do snapshotu jako `status: UNCLEAR`.

**Timing:** typicky 30-60s pro 60 paralelních calls. Pokud > 120s, retry s exponential backoff (max 3×) per failure handler #24.

## 21. Monday brief draft formats

Brief draft writer per channel (Slack / Gmail):

```python
# Slack Connect channel draft (per Smartin Connect setup)
mcp__a6f5a044-5754-4aec-b1d2-0fe84fd753f9__slack_send_message_draft(
    channel="#klient-{slack_channel_name}",
    text=brief_text,  # plain text, no markdown table (Slack render)
    metadata={
        "smartin_type": "monday_brief",
        "iso_week": "2026-W25",
        "workspace_id": "bf8bf432-..."
    }
)

# Gmail draft
mcp__c1290c8d-d03d-434c-a8c4-d9952b768b9f__create_draft(
    to="karel@legalsystems.cz",  # team.sponsor.email
    subject="Pondělí 22. 6. · Smartin · LS",
    body=brief_text,  # plain text email format
    labels=["Smartin × {company_name}"]
)
```

**Formátování per channel:**
- Slack: monospace blok pro číselné tabulky (zarovnání), žádný markdown
- Gmail: plain text, žádné HTML (mobile-friendly)
- Oba: ASCII safe (žádné Unicode emoji v subject — jen v body)

**Idempotence:** plugin kontroluje, jestli draft pro daný `iso_week` už existuje (přes metadata search). Pokud ano, replace, ne duplicate.
