# smartin-beta

Smartin AI decision agent jako Cowork plugin. MVP balení čtyř skills, které dohromady tvoří onboarding agenta a denního průvodce pro klienty Smartinu i Smartin tým sám.

Po beta fázi nahradí čistý MCP server, který poběží v jakémkoliv MCP-kompatibilním klientovi (Cowork, Claude desktop, ChatGPT desktop, Cursor a další). Plugin je most do první distribuce.

## Co plugin obsahuje

### Čtyři skills

| Skill | Kdy se aktivuje | Co dělá |
|---|---|---|
| `smartin-methodology` | „Smartin", „decision system", „Reality → Interpretace → Next step", „decision memo", „hypothesis", „case type", „maturity audit" | Vynucuje Smartin tón, strukturu, lexikon, decision loop |
| `smartin-api-access` | „Smartin API", „case-config", „hypothesis", „researchFields", „case-result", „/workspace/", „dev.mjobs.blog", „smartin.work", „token", „401", „Vyhodnotit" | Recepty pro token extraction, klíčové endpointy, field mapping, common traps |
| `smartin-onboarding-guide` | „onboarding", „nový klient", „playbook", „Legal Systems", „první session", „Fáze 0", „Pre-flight", „Aktualizovat poprvé" | 7-fázový onboarding playbook se třemi role mody (self-serve / facilitator / internal) |
| `smartin-failure-handlers` | „Aktualizace nedetekovala", „klient nemá HubSpot", „401", „workspace prázdný", „klient se zasekl", „detekce nesedí" | Branchové handlery pro 10 nejčastějších failure modes |

Skills se aktivují automaticky podle obsahu rozhovoru. Není potřeba je vyvolávat explicitně.

## Pro koho je plugin určen

Plugin běží ve třech operačních modech podle identity uživatele.

### Self-serve mode

Externí klient (např. matula@legalsystems.cz). Plugin vede uživatele 7 fázemi onboardingu, generuje deliverables, navrhuje další krok. Facilitator nepřítomen. Default mode.

### Facilitator mode

Smartin konzultant (např. martin.soucek@smartin.work). Plugin ukazuje portfolio všech klientů, jejich stage v playbooku, blokace, kdo dnes potřebuje pozornost. Generuje ranní fronta „kdo a proč".

### Internal mode

Smartin tým (@smartin.work nebo @24snails.com). Self-serve na vlastní firmě + facilitator nad klienty + produktová rovina s cross-tenant agregovanými signály. Tato rovina není dostupná v jiných modech.

## Instalace

Beta fáze = manuální distribuce přes Slack.

1. Stáhnout `smartin-beta.plugin` (zip soubor).
2. V Coworku otevřít Settings → Plugins → Install from file.
3. Vybrat stažený zip.
4. Cowork plugin nainstaluje a aktivuje skills.
5. Při prvním použití plugin zjistí identitu uživatele a nastaví příslušný mode.

## Závislosti

Plugin neobsahuje vlastní MCP server. Spoléhá na to, že uživatel má v Coworku napojeny vlastní MCPs pro:

- Slack (pro #klient-* kanály, interní komunikaci)
- HubSpot (CRM, pipeline, deals)
- Google Calendar (kalendář, weekly review invites)
- Gmail (klientská komunikace)
- Google Drive (dokumenty)
- Google Analytics (pokud relevantní pro Marketing oblast)

Smartin API access probíhá přes browser tab v Smartin appce (smartin.work/chat nebo dev.mjobs.blog), kde plugin extrahuje token z React fiber tree.

## Autor

Smartin (martin.soucek@smartin.work)
