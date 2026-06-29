---
name: smartin-snapshots
description: Týdenní snapshots Mapy firmy s textem + čísly. Use this skill kdykoli plugin potřebuje vytvořit, načíst nebo srovnat snapshot stavu klientova workspace. Triggery — neděle 23:55 weekly snapshot, Phase 0.7 baseline, quarterly reset, on-demand "udělej snapshot teď". Skill řídí storage v 09_Klienti/Smartin × {klient}/snapshots/, idempotenci časovým stamp, diff vůči předchozímu snapshotu, a poskytuje data pro /smartin-status a Monday brief.
---

# Smartin Snapshots

Plugin po každé Aktualizaci přepíše stav v Smartin app DB. Bez snapshots klient ani konzultant nevidí **vývoj v čase** — jen aktuální okno. Tento skill řídí trvalý zápis snapshots do `09_Klienti/Smartin × {klient}/snapshots/` plus diff logic pro trend čtení.

## 1. Tři typy snapshots

| Typ | Kdy | Trigger | Soubor |
|---|---|---|---|
| Baseline | Po Phase 0.7 standardní cesta + quarterly reset | Auto po `evaluate_workspace` | `{YYYY}_Q{N}_baseline.yaml` |
| Weekly | Neděle 23:55 (ISO week boundary) | Scheduled task `smartin-snapshot-weekly` | `{YYYY-WW}.yaml` |
| On-demand | Klient řekne „udělej snapshot teď" | Explicit klientův command | `{YYYY-MM-DD_HH-MM}_ondemand.yaml` |

## 2. Storage struktura

```
09_Klienti/Smartin × {klient}/snapshots/
├── _index.yaml
├── 2026_Q3_baseline.yaml
├── 2026-W25.yaml
├── 2026-W26.yaml
└── archive/
    ├── baseline_Q1_2026.yaml
    └── baseline_Q2_2026.yaml
```

## 3. Snapshot schema

Klíčové fields:
- `captured_at`, `trigger`, `workspace_id`, `plugin_version`, `iso_week`
- `overview` per oblast (result_group, severity_avg, case_count, top_signals)
- `cases[]` per case: severity, status (DETECTED/NOT_DETECTED/UNCLEAR), result_group, branch, `ai_generated_result` (full text), `metrics` (open_deals, pipeline_value_kc, atd. + delta_vs_prev_snapshot), source_query_resolved, source_types
- `prefetch_metrics` (raw čísla z connectorů)
- `maturity_answers` (kompletní stav RFs)
- `prev_snapshot`, `diff_summary` (cases_improved, cases_worsened, severity_avg_delta)

## 4. Workflow per typ

**Baseline** (po Phase 0.7) — auto po evaluate_workspace, archive starou baseline pokud quarter změnil.

**Weekly** (neděle 23:55) — idempotence časovým stamp, pokud existuje skip. Evaluate_workspace + poll fresh data + 60 case_results paralelně + write + diff.

**On-demand** — klient řekne „udělej snapshot teď", plugin potvrdí, spustí build_full_snapshot s trigger=on_demand.

**Quarterly reset** — 1. pondělí Q1/Q2/Q3/Q4: archived starou baseline, vytvoří novou.

## 5. build_full_snapshot — co plugin tahá

```python
def build_full_snapshot(workspace_id, trigger):
    overview = mcp__smartin__smartin_get_overview(workspace_id)
    cases = await get_all_case_results(workspace_id)  # 60 paralelních calls
    prefetch = get_prefetch_metrics(workspace_id)
    answers = get_maturity_answers(workspace_id)
    return {
        "captured_at": now_iso(),
        "trigger": trigger,
        "workspace_id": workspace_id,
        "plugin_version": "0.11.0",
        "overview": overview,
        "cases": cases,
        "prefetch_metrics": prefetch,
        "maturity_answers": answers,
    }
```

## 6. diff vůči předchozímu

```python
def compute_diff(prev, current):
    cases_improved = []
    cases_worsened = []
    for case_key in current.cases:
        prev_sev = prev.cases.get(case_key).severity
        curr_sev = current.cases[case_key].severity
        if curr_sev < prev_sev - 5:
            cases_improved.append(case_key)
        elif curr_sev > prev_sev + 5:
            cases_worsened.append(case_key)
    return {
        "cases_improved": cases_improved,
        "cases_worsened": cases_worsened,
        "severity_avg_delta": avg_delta(prev, current),
    }
```

## 7. Failure modes

- Snapshot už existuje pro daný ISO week → No-op (idempotent)
- `evaluate_workspace` 500 → Failure handler #24
- Drive write 403 → Failure handler #14 nebo #21
- 60 case calls timeout > 120s → retry exponential backoff (max 3×)

## Reference

- Spec: `04_Produkt/smartin-beta-plugin/signaly/plugin_v0_11_snapshots_timeseries_15062026.md`
- `smartin-monday-brief` skill — čte snapshots pro brief generation
- `/smartin-status` command — čte snapshots pro trend view
- `smartin-api-access` §20 — get_all_case_results recipe
