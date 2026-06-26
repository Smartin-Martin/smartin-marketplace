# Repository — where ideas live and how to handle them

Compass stores everything in plain files in a folder the user controls. No backend, no database. This keeps it local-first and private.

## Default location

On first use, create a `napadovnik/` folder inside the working folder the user has connected, without asking. Do not prompt for a folder on first run. If the user later asks to move it, or wants a shared folder for a team, accommodate that.

## Structure

One idea is one folder.

```
napadovnik/
  _index.md                      # one line per idea, with maturity at a glance
  napady/
    skoleni-uctetni/
      nazev.md                   # human title + one-sentence description
      podnety.md                 # append log: date, source, raw text
      kosticky.yaml              # coverage per tile (hidden from the user)
      stav.yaml                  # maturity, last nudge, next suggested step
  _archiv/                       # set-aside or dropped ideas
```

## `_index.md`

One line per idea so you can scan the whole repository fast. Keep it current whenever an idea changes.

```
- skoleni-uctetni — Školení pro účetní | zrání: 2/6 tiles | poslední podnět: 2026-06-11
- api-pro-mzdarky — API pro mzdové účetní | zrání: 4/6 tiles | poslední podnět: 2026-06-09
```

## `podnety.md` (append-first)

Every caught fragment goes here first, before any sorting.

```
## 2026-06-11 · konverzace
"Co kdybychom dělali školení pro účetní v malých firmách."

## 2026-06-12 · odkaz (https://example.com/clanek)
Článek o tom, že malé účetní firmy bojují s nábor. Možný problém k řešení.

## 2026-06-13 · citace zákazníka
"Nemáme čas zaškolovat juniory, všechno padá na seniory."
```

## `kosticky.yaml` (hidden coverage map)

Never shown to the user. Drives the confrontation and the nudge.

```yaml
zakaznik:   { pokryti: castecne, poznamka: "účetní v malých firmách, 2 zmínky" }
problem:    { pokryti: dobre,    poznamka: "zákazník popsal sám, čas seniorů" }
produkt:    { pokryti: prazdne }
obchod:     { pokryti: prazdne }
cena:       { pokryti: prazdne,  poznamka: "tady chybí, vhodné šťouchnout" }
navratnost: { pokryti: prazdne }
```

`pokryti` is `prazdne`, `castecne`, or `dobre`. This marks where the idea has real input, not whether anything is confirmed.

## `stav.yaml`

```yaml
nazev: "Školení pro účetní"
slug: skoleni-uctetni
zrani: "2/6 tiles"
posledni_podnet: 2026-06-13
posledni_souch: 2026-06-08
dalsi_krok: "Zjistit, jestli by za školení někdo zaplatil."
stav: aktivni      # aktivni | archiv
```

## Merge rule

Before creating a new idea folder, read `_index.md` and judge whether the new fragment belongs to an existing idea. If it clearly does, append to that idea's `podnety.md` and update its maps. If it is borderline, ask the user in one line. Never duplicate, never fuse two distinct ideas.

## Privacy

Everything here is local to the user's folder. Nothing is uploaded anywhere on its own. If the user has connected Smartin and asks to sync, that is the only path data leaves this folder, and only using the Smartin tools present in the session.
