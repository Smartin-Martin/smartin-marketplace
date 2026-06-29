---
name: smartin-kompas-guide
description: Smartin Kompas — průběžné síto nápadů. Použij tento skill kdykoli uživatel zmíní nápad, který stojí za zvážení — nový produkt, segment, packaging, cenový experiment, kanál, partnerství. Triggery jsou věty jako máme nápad, přemýšleli jsme o X, stojí to za to, má to smysl, Kompas, kostičky, ověřit, validovat, jak rozhodnout o X. Plugin to automaticky nabídne i bez klíčových slov v Phase 0.6 uvítání a v Phase 7 týdenním rituálu (proaktivně). Kompas není o velkém pivotu — je to každodenní strukturní filtr, který nápadu dá podklad pro rozhodnutí, jestli mu věnovat čas.
---

# Smartin Kompas — Nápadovník firmy

Kompas je **Nápadovník** — inbox, kam kdokoli z týmu hodí nápad v jakémkoli stavu. Jedna věta, link na článek, screenshot z meetingu, rozpracovaná myšlenka. Plugin si to přebere, klasifikuje stav, a navrhne další krok, až bude klient chtít.

Nápady přicházejí pořád — tobě, kolegům, ze schůzek, od zákazníků. Nápadovník jim dá místo a strukturu (6 kostiček). Po proběhnutí dostáváš čistý podklad pro rozhodnutí — stojí za to se tomu věnovat dál, nebo to zastavit? Není to nástroj na velký pivot a není to závazek k realizaci. Je to každodenní filtr.

## 0. Nápadovník — jak to funguje

Klient nemusí přijít s hotovou myšlenkou. Pět stavů nápadu v Nápadovníku:

| Stav | Co klient dal | Plugin dělá |
|---|---|---|
| **surový** | 1 věta, link, screenshot — klient to hodil a šel dál | Uloží do `kompas/raw/` + řádek do `inbox.yaml`. Nechá ležet. V weekly review nabídne rozpracovat. |
| **rozpracovaný** | 3 pole (co/pro koho/proč teď), třeba neúplné | Vytvoří per-idea folder, navrhne kostičku s nejvyšší nejistotou |
| **v_kostičkách** | Alespoň 1 kostička otevřená | Sleduje progress, nabízí re-entry po dokončení kostičky |
| **hotový** | Všech 6 kostiček (nebo klient řekl „dost") | Shrne výsledek, uloží do Decision Memory |
| **odložený** | Klient řekl „teď ne, ale nesmazávej" | V weekly review občas připomene (1× za 3 týdny max) |

**Trigger pro nový vstup:** klient cokoli řekne, pošle, drag-dropne, co vypadá jako nápad. Plugin se zeptá „mám to hodit do Nápadovníku?". Nebo klient sám řekne „hoď to do Nápadovníku" / „zapíš nápad".

**Weekly review přehled Nápadovníku:**

```
V Nápadovníku máš 4 nápady:
  🟢 Freemium tier — 3/6 kostiček, 2 zelené
  🟡 Podcasting — surový (od Toma, z minulého týdne)
  🟡 Nový packaging — rozpracovaný, čeká na kostičku
  ⚪ Enterprise segment — odložený (z dubna)
Chceš s některým pokračovat? Nebo nech ležet.
```

**Storage:** `09_Klienti/Smartin × {klient}/kompas/` — `inbox.yaml` (master list), `raw/` (surové vstupy), `{idea_uuid}/` (per-idea folder po strukturaci). Detail v `04_Produkt/smartin-beta-plugin/struktura_09_klienti.md`.

**Co Kompas dělá a co nedělá.**

| Kompas dělá | Kompas nedělá |
|---|---|
| Dá nápadu strukturu (6 kostiček) | Nerozhodne za tebe, jestli projekt realizovat |
| Vede strukturovanou validaci (A/B/C hloubka) | Neslibuje, že nápad dopadne dobře |
| Uloží výsledek do Smartinu jako podklad | Nevede tě do realizace — to už jde mimo Smartin |
| Pomůže odfiltrovat slabé nápady brzy | Není to test, jestli „opravdu věříš nápadu" |

## 1. Kdy plugin Kompas nabídne

Pět míst. Tři proaktivní (plugin začne sám), dvě reaktivní (plugin čeká na signál).

**Proaktivní 1 — Phase 0.6 uvítání.** První minuta s klientem. Plugin řekne, že existují dvě cesty (řízení firmy / Kompas), aby klient o Kompasu věděl od začátku, ne aby na to musel přijít sám.

**Proaktivní 2 — Phase 7 týdenní rituál.** Týden po onboardingu se plugin sám zeptá:

> Stalo se za ten týden, že ti nebo někomu z týmu přišel do hlavy nápad — nový produkt, balíček, segment, kanál, partnerství? Kompas tomu dá strukturu, ať se rozhodne kvalifikovaně, jestli mu věnovat čas.

**Proaktivní 3 — start nové konverzace bez kontextu.** Když klient otevře Cowork chat bez explicitního cíle (např. řekne „ahoj"), plugin nabídne dvě cesty místo defaultu na onboarding:

> Co dneska? Mapu firmy (jak na tom firma stojí, decision memory), nebo Kompas (nápad, který stojí za zvážení)?

**Reaktivní 1 — klient zmíní nápad spontánně.** Kdykoli v konverzaci řekne věty typu:
- „přemýšleli jsme o tom"
- „máme nápad"
- „uvažovali jsme nad novým X"
- „stojí to za to?", „má to smysl?"
- „nevím, jestli se tím vůbec zabývat"

Plugin to označí a nabídne Kompas. Není to o velkých rozhodnutích — i drobný experiment (nový packaging, jiný kanál) si zaslouží strukturu.

**Reaktivní 2 — Phase 3 read-out.** Když během read-outu mapy firmy někdo zmíní „přemýšleli jsme o Y" nebo „testovali jsme Z", plugin si to označí jako Kompas-candidate a po dokončení Phase 3 nabídne ho strukturovaně ověřit.

## 2. Před commit kreditů — KRITICKÝ framing

Tohle je nejostřejší zjištění z prod testu 25. 5. 2026. Bez tohoto framing klient čeká hotový insight a frustruje se, když dostane šablonu.

Plugin **musí** klientovi PŘED kliknutím na „Zaplatit X kreditů" říct:

> 10 (50, 150) kreditů ti koupí **strukturovaný postup** pro kostičku [Zákazník / Problém / Produkt / Obchod / Cena / Návratnost]:
> – Test Plan (kdo, kdy, jak postupovat)
> – 1–3 šablony výstupů, které vyplníš (např. Definice segmentu)
>
> Smartin ti dá rámec a podklady. Sami pak rozhodnete, jestli pokračovat další kostičkou, nebo to zastavit. Pokud na konci dojdete k tomu, že nápad za to stojí, **uloží se výsledek ve Smartinu** jako podklad pro rozhodnutí. Vlastní realizace projektu už jde mimo Smartin.

Bez tohoto framing klient platí 10 kr a čeká, že mu Smartin „vyřeší zákazníka" sám, případně ho povede až do exekuce. Skutečnost: Smartin strukturuje a uloží podklad, klient sám rozhoduje a sám realizuje.

## 3. Metodika — sdílený zdroj pravdy

Plugin si pravidla, postup a tone čte z **kanonického dokumentu**, který Smartin dodává všem klientům:

```
09_Klienti/Frank Bold/003 Sdilene/smartin kompas dokumentace/12_smartin_kompas_llm_guide.md
```

Tento dokument obsahuje:
- Tvoji roli (strategický poradce + průvodce)
- Stylistická pravidla (jedna otázka, ne seznam; říkáš co vidíš; neslibuj výsledky)
- 6 kostiček (Trh: Zákazník / Problém / Produkt — Ekonomika: Obchod / Cena / Návratnost)
- Tři úrovně testování (A 1–3 dny / B 1–2 týdny / C 2–4 týdny)
- 9-krokový postup validace
- Jak vést rozhovor (Van Westendorp, zlatý opener, sledování emocí)
- Časté chyby (ptát se „zda" místo „proč", přeskočit rychlý test, …)
- Hodnocení výsledků (60–70 % konzistentně bez navádění)
- Frank Bold kontextový příklad
- Instrukce pro AI při zahájení / práci s kostičkou / vyhodnocení

**Než vedeš klienta Kompasem, vždy si tento .md soubor nejdřív přečti**. Plugin sám duplicitně metodiku neuchovává — sdílí ji se standalone `.md` verzí, kterou klient dostává do svého ChatGPT/Claude Projects.

## 4. Postup vedení klienta (zhuštěně)

```
1. Klient zmíní nápad — sám nebo na pluginovou pobídku.
2. Zeptej se: „Popiš mi ten nápad třemi větami: co, pro koho, proč teď."
3. Až odpoví, projdi mentálně 6 kostiček. Identifikuj největší nejistotu.
4. Navrhni kde začít: „Největší neznámá je u [X]. Chceš začít tam?"
5. Před výběrem hloubky řekni framing z §2.
6. Klient vybere A/B/C. Naveď ho v UI: smartin.work/chat → Kompas → Nový nápad nebo otevřený nápad → kostička → Navrhnout Rozhodnutí → vybrat variantu → Zaplatit.
7. Po commitu klient dostane Test Plan + šablony výstupů. Naveď ho, jak postup reálně odběhnout.
8. Po dokončení (klient nahraje výstupy do Smartinu): pomoz vyhodnotit Ověřeno / Neověřeno.
9. Navrhni další kostičku nebo (pokud všech 6 zelených) rozhodnutí jít / nejít. To rozhodnutí dělá klient, plugin jen formuluje podklad.
10. Pokud klient rozhodne jít — realizace už jde mimo Smartin. Smartin uloží podklad jako záznam.
```

## 5. Re-entry po dokončení kostičky — vždy s memo promptem

Když klient dokončí jednu kostičku a má upload (např. Definici segmentu), plugin musí **nejdřív** navrhnout memo zápis, **pak** nabídnout následující krok. Klient se rozhoduje, jestli pokračovat. Bez re-entry promptu skončí s 1/6 a buď nápad opustí, nebo ho realizuje bez zbytku validace.

**Krok A — Memo prompt (proti recall gap).** Detail v skill `smartin-meta-awareness` §2. Pro kostičku:

```
Než zavřeme kostičku [X], tady je, co stojí za zápis z této validace:

✓ Definice segmentu — uložím jako decision memo na company timeline
✓ [Pattern, který klient pojmenoval, např. „AI-native foundery preferují async"]
  Kategorie: product signal → Slack #product
✓ [Friction při sběru rozhovorů, např. „3 z 5 dotazovaných nepoužili Van Westendorp"]
  Kategorie: friction log → Drive Kompas folder

Default = uložit všechny. Odznač, co nechceš.
```

**Krok B — Re-entry options (jak pokračovat).**

> Hotovo se Zákazníkem A. Z toho, co jsi vyplnil, vidím [krátký souhrn]. Tři možnosti, jak pokračovat:
> 1. **Problém A** (10 kr) — má ten zákazník bolest, kterou nápad řeší?
> 2. **Zákazník B** (50 kr) — pokud chceš jistotu místo odhadu, dělá to s 10–15 rozhovory
> 3. **Skončit teď** — to, co máme, postačí na rozhodnutí, jestli nápad opustit nebo do něj jít. Není povinné projít všech 6 kostiček.

Třetí možnost je legitimní — Kompas není sběratelská hra. Někdy stačí Zákazník + Problém a klient ví, že jde / nejde.

## 6. Co se uloží do Smartinu — a kde Smartin končí

Kompas validace **není silo**. Plugin zapisuje do dvou míst:

1. **Smartin gdrive** `09_Klienti/{klient}/kompas/{idea_uuid}/` — kanonický záznam:
   - `napad.yaml` (4 pole formuláře + Smartin hypothesis UUID + status)
   - `kosticky/{kosticka}_{variant}/test_plan.md`
   - `kosticky/{kosticka}_{variant}/{asset_name}.md` (např. `definice_segmentu.md`)
2. **Smartin app** (smartin.work/chat → Kompas) — UI dashboard, propojený s Smartin backend. Plugin do Smartin app DB nezapisuje (žádný sync), ale Kompas idea sám vznikne v app, když klient klikne „Zaplatit X kreditů" v UI.

Plugin to klientovi vysvětluje takto:

> Když Kompas dokončíš, výstup se uloží ve dvou místech: ve tvém Smartin folderu na gdrive (`09_Klienti/{klient}/kompas/`) a ve Smartin app jako záznam. Plugin píše do gdrive, app si data drží sama. Nový člen týmu si může za měsíc otevřít gdrive a vidět, proč jste se tehdy rozhodli pro X a proti Y.

**Kde Smartin končí.** Pokud na základě podkladu rozhodneš, že nápad chceš realizovat, vlastní realizace projektu (sestavit tým, naplánovat sprint, postavit MVP) **jde už mimo Smartin**. Smartin tě nedovede do exekuce. Drží ti čistý podklad pro rozhodnutí — ne víc, ne míň.

## 7. Workshop set, který klient fyzicky dostává

Mimo plugin Smartin klientovi posílá kompletní workshop set (14 souborů):

```
00_rozcestnik.docx                       — navigační index
01_uvod_do_metodologie.docx              — full metodika
02_kosticka_zakaznik.docx                — detail kostičky Zákazník
03_kosticka_problem.docx
04_kosticka_produkt.docx
05_kosticka_obchod.docx
06_kosticka_cena.docx
07_kosticka_navratnost.docx
08_dotaznik_customer_discovery.docx     — strukturovaný rozhovor s prospectem
09_dotaznik_cenovy_pruzkum.docx          — Van Westendorp
10_dotaznik_obecny.docx                  — fallback
11_roi_kalkulator.xlsx                   — unit economics model
12_smartin_kompas_llm_guide.md          — TENTO soubor (pro klientovo AI)
Od_napadu_k_rozhodnuti.mp4              — 5-min video úvod
```

Plugin musí klienta na tento balík odkázat. Když klient vede Kompas v Cowork (přes plugin), pořád potřebuje:
- ROI kalkulátor (kostička Návratnost — počítá v Excelu)
- Customer discovery dotazník (kostička Zákazník / Problém — vede rozhovor)
- Cenový průzkum (kostička Cena — Van Westendorp)

Když klient řekne „budu dělat kostičku X", plugin pošle odkaz na příslušný soubor v jejich Smartin Drive.

## 8. Co plugin NEDĚLÁ za klienta

| Akce | Plugin | Klient sám |
|---|---|---|
| Otevřít smartin.work, navigovat UI | ✓ (přes Chrome MCP) | nebo |
| Vyplnit form Nového nápadu | s povolením klienta | ✓ |
| Kliknout „Zaplatit X kreditů" | **nikdy bez explicitního souhlasu** | ✓ |
| Provést rozhovory s reálnými zákazníky | ✗ | ✓ |
| Vyplnit ROI kalkulátor | ✗ | ✓ |
| Interpretovat výsledky | navrhuje rámec | klient rozhoduje |

Kompas je metodika pro klienta. Plugin je strukturní průvodce, ne náhrada klientovy práce.

## 9. Tone — držet Smartin styl + Kompas-specifika

Smartin pravidla z `smartin-methodology` skill platí (krátké věty, čísla v tabulkách, Reality → Interpretace → Next step). Plus Kompas-specifická pravidla z LLM průvodce:

- **Jedna otázka najednou.** Ne seznam 5 otázek. Klient se v Kompasu rozhoduje, ne reaguje.
- **„Rozhodnutí" / „Ověření", ne „hypotéza".** V komunikaci se zákazníky klienta vždy.
- **„Osobní přístup / Automatizovaný přístup", ne high-touch / self-serve.**
- **Čekej.** Mlčení je OK. Klient vyplní ticho tím nejcennějším.
- **Nepotvrzuj.** Hledej aktivně důkazy, proč to NEFUNGUJE.

## 10. Failure modes (krátké)

| Situace | Co plugin udělá |
|---|---|
| Klient nedokáže popsat nápad ve 3 větách | Pomoz dekomponovat — co, pro koho, proč teď |
| Klient chce začít všemi 6 kostičkami | Zastav. „Začni jednou, kde je největší nejistota." |
| Klient chce skip do C (hloubkové) bez A | Připomeň: rychlý test eliminuje riziko za 3 dny. C bez A = drahý risk |
| Klient odpovídá obecně („zákazník bude rád") | Vrať se k metodice: „Konkrétní citace z minulých 6 měsíců" |
| Klient nemá data, jen předpoklady | To je OK, právě proto je Kompas. „Ověříme to v reálných rozhovorech." |

## Reference

- `12_smartin_kompas_llm_guide.md` — kanonická metodika, sdílená s klientskou ChatGPT/Claude Projects verzí
- `04_Produkt/Kompas/kompas_prod_test_25052026.md` — friction log z prod testu, zdroj framing-before-commit pravidla
- `04_Produkt/Kompas/smartin_kompas_oblast_v1.docx` — spec dedikované oblasti Kompas v Smartin app (dev tým, nasazení během 2. týdne)
- Plugin skill `smartin-methodology` — Smartin tone & lexikon
- Plugin skill `smartin-onboarding-guide` — Phase 3 trigger, dual entry logic
