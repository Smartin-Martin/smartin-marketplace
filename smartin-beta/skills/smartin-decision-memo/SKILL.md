---
name: smartin-decision-memo
description: Zápis rozhodnutí jako structured memo. Use this skill kdykoli klient nebo konzultant udělá důležité rozhodnutí a chce ho zachovat — "zapiš to", "decision memo", "ulož rozhodnutí", "tohle si pamatuj". Plugin zachytí strukturu Reality → Interpretace → Next step ze Smartin metodiky a uloží do Drive 09_Klienti/Smartin × {Klient}/decision_memos/.
---

# Smartin Decision Memo

Rychlý strukturovaný zápis rozhodnutí. Žádné dlouhé eseje — Reality, Interpretace, Next step, tečka.

## Kdy spustit

Plugin spouští skill, když klient nebo konzultant:
- explicitně řekne „zapiš to", „decision memo", „ulož rozhodnutí", „tohle si pamatuj"
- ukončí významnou debatu jasným závěrem („tak to teda uděláme X", „necháme to být")
- v Phase 7 weekly review konsoliduje týden (digest = několik memos sloučených)

Pravidlo: ne každé rozhodnutí potřebuje memo. Default = neptej se. Triggerové fráze jsou explicit signál, že klient memo chce.

## Struktura memo

```markdown
# {Stručný název rozhodnutí, max 8 slov}

**Datum:** {YYYY-MM-DD HH:MM}
**Klient:** {company_name}
**Owner:** {kdo to rozhodl — sponsor / daily owner / konzultant}
**Oblast:** {Sales / Marketing / CSM / Operations / Customer & trh / Product / Cross-area / Smartin operations}

## Reality

{1-3 věty — co se reálně stalo, jaká data, jaký konkrétní stav firmy.}

## Interpretace

{1-3 věty — co to znamená pro firmu. Kde je leverage, kde je risk.}

## Next step

- {konkrétní akce, owner, deadline}
- {další akce pokud relevantní}
```

## Workflow

1. Plugin detekuje trigger (frází nebo end-of-debate situace).
2. Plugin zopakuje kontext: „Slyšel jsem rozhodnutí: {1 věta shrnutí}. Mám zapsat memo?"
3. Klient potvrdí (default = ano, klient může říct „ne, ještě ne").
4. Plugin vede strukturou — krátké otázky:
   - „Reality: co se reálně stalo?" (1-2 věty od klienta)
   - „Interpretace: co to znamená?" (1-2 věty)
   - „Next step: co teď?" (akce + owner)
5. Plugin sestaví markdown per template.
6. Plugin uloží do `09_Klienti/Smartin × {Klient}/decision_memos/{YYYY-MM-DD}_{slug}.md`.
7. Plugin pošle Slack notif do `#klient-{slack_channel_name}` (nebo email per `communication_channel`).
8. Plugin update `session_state.yaml`: `last_action: "Decision memo: {název}", last_action_at: {timestamp}`.

## Co plugin NEDĚLÁ

- Nepíše memo bez triggeru. Klient memo žádá, ne plugin nutí.
- Nepřepisuje existující memo. Každé memo je immutable.
- Nezapisuje do HubSpot Notes default (gdrive je primary od v0.6+).
- Neptá se na MVA upload v memo flow (memo je decision, ne důkaz).

## Reference

- Smartin metodika Reality → Interpretace → Next step: `smartin-methodology`
- Storage struktura: `09_Klienti/Smartin × {Klient}/decision_memos/`
- Weekly digest pattern: `smartin-onboarding-guide` Phase 7
- Slack notif format: `smartin-api-access` §13 (draft-first rule)
