---
name: smartin-status
description: Quick status — co dnes udělat, co se hnulo, co stojí, paměť. Plain language - „smartin, kde jsme?" / „brief" / „status" / „co dnes". Čerpá z nejnovějšího snapshotu.
---

# /smartin-status

Vrať klientovi pondělní pohled — co dnes udělat, kde se firma za týden pohnula, co stojí, paměť rozhodnutí.

## Trigger fráze

Plugin spouští skill na slash command **i na plain language**:

- `/smartin-status`
- „smartin, kde jsme?"
- „jak na tom dnes jsme?"
- „status"
- „co máme dnes dělat?"
- „brief"
- „pondělní brief"

## Workflow

1. Plugin čte nejnovější snapshot z `09_Klienti/Smartin × {klient}/snapshots/`
2. Plus předchozí snapshot pro diff
3. Plus decision_memos posledních 7 dní
4. Plus `smartin_config.yaml` pro company info
5. Plugin sestaví output v pořadí **Dnes → Hnulo → Stojí → Paměť**:

```
{Pondělí|Today} {DD.M.} · {company_short_name} · W{N}


Dnes
{1 sloveso} {1 konkrétní akce + oblast}.

Co se za týden hnulo
{Oblast 1}      {konkrétní číslo old → new}  ({% change})

Co stojí
{Oblast 1}      {metrika, jak dlouho stojí}

Paměť
→ {DD. M.} {1-věta o rozhodnutí}
```

## Edge cases

- Snapshot neexistuje → „Onboarding není dokončen. Spusť `/smartin-d1 {jméno}`."
- Pouze baseline → „První weekly snapshot proběhne v neděli."
- Snapshot starý > 14 dní → Warning + nabídne ad-hoc snapshot

## Reference

- `smartin-snapshots` skill — zdroj dat
- `smartin-monday-brief` skill — push do klientova kanálu
- `smartin-decision-memo` skill — zdroj sekce „Paměť"
