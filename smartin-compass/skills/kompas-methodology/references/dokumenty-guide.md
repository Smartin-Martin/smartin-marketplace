# Dokumenty-guide: generování workshopového setu

Vše, co potřebuješ k vygenerování celého Smartin Compass workshopového setu pro klienta.

## Kdy generovat

Generuj plný set, když uživatel spustí `/kompas` a dodá dost kontextu o firmě a nápadu. Sběr kontextu řeší command, tento soubor pokrývá samotné generování.

## Plný set (13 dokumentů)

| # | Soubor | Typ | Účel |
|---|---|---|---|
| 00 | `00_rozcestnik.docx` | Word | Navigační index, přehled dokumentů a kdy je použít |
| 01 | `01_uvod_do_metodologie.docx` | Word | Úvod do metodiky, co Compass je a jak ho použít |
| 02 | `02_pilir_zakaznik.docx` | Word | Zákazník, A/B/C testy plus TAM/SAM/SOM |
| 03 | `03_pilir_problem.docx` | Word | Problém, A/B/C testy |
| 04 | `04_pilir_produkt.docx` | Word | Produkt, A/B/C testy |
| 05 | `05_pilir_obchod.docx` | Word | Obchod, A/B/C testy (akvizice plus dodání) |
| 06 | `06_pilir_cena.docx` | Word | Cena, A/B/C testy |
| 07 | `07_pilir_navratnost.docx` | Word | Návratnost, A/B/C testy plus ROI logika |
| 08 | `08_dotaznik_customer_discovery.docx` | Word | Dotazník pro customer discovery |
| 09 | `09_dotaznik_cenovy_pruzkum.docx` | Word | Dotazník pro cenový průzkum |
| 10 | `10_dotaznik_obecny.docx` | Word | Obecná šablona dotazníku |
| 11 | `11_roi_kalkulator.xlsx` | Excel | ROI kalkulátor (vstupy, jednotková ekonomika, doba návratnosti) |
| 12 | `12_smartin_compass_llm_guide.md` | Markdown | Průvodce pro nahrání do AI asistenta pro samostatné použití |

## Struktura dokumentu pilíře (02 až 07)

Každý dokument pilíře drží tuto strukturu. Příklady přizpůsob firmě a nápadu uživatele.

```
# [Název pilíře]: [Firma uživatele]

## Proč na tomto pilíři záleží
2 až 3 věty, proč tento předpoklad pro jejich konkrétní nápad záleží.

## Ukazatel ověření
Jak vypadá „ověřeno", konkrétní signál nebo práh.

## Test A: Rychlý (1 až 3 dny)
### Co děláme
### Postup
### Výstup
### Kritéria úspěchu

## Test B: Standardní (1 až 2 týdny)
## Test C: Hloubkový (2 až 4 týdny)

## Výsledek
### OVĚŘENO: co to znamená
### NEOVĚŘENO: co to znamená (úpravy nebo kdy zastavit)
```

## Rozcestník (00)

Jednostránkový přehled. Co je Compass (2 věty). Tabulka 6 pilířů (název, klíčová otázka, stav prázdný k vyplnění). Jak pracovat s dokumenty. Doporučené pořadí (začni pilířem s největší nejistotou).

## Úvod do metodologie (01)

Proč Compass pro tuto firmu. 6 pilířů, plná tabulka. 3 úrovně testů. Jak vyhodnocovat (OVĚŘENO/NEOVĚŘENO). Jak začít, konkrétní první kroky pro tohoto klienta.

## Dotazníky (08 až 10)

08, Customer Discovery. Strukturuje rozhovory pro Zákazníka a Problém. Sekce: úvod, kdo jste a co řešíte, jak řešíte teď, co vás komplikuje, reakce na náš návrh, závěr.

09, Cenový průzkum. Pro pilíř Cena. Sekce: kontext, 4 Van Westendorp otázky, ochota platit (měsíčně, ročně, jednorázově), srovnání s alternativami.

10, Obecný. Flexibilní šablona. Téma průzkumu, cílová skupina, 10 prázdných otázek s návodem.

## ROI kalkulátor (11)

Excel, 3 listy. Vstupy (segment, průměrná cena, objem za rok, náklad na akvizici, náklad na dodání, míra odchodů). Jednotková ekonomika (příjem na zákazníka, náklad na zákazníka, marže, doba návratnosti, hodnota zákazníka, poměr hodnoty k nákladu). Výhled (měsíční kumulativní příjem vs. náklad, zvýrazněný bod zvratu, graf).

## LLM průvodce (12)

Markdown k nahrání do AI asistenta. O Compasu (2 odstavce). 6 pilířů (tabulka). Jak pomáhat uživateli. Pravidla hodnocení. Příklady otázek. Sekce „Kontext klienta" k vyplnění před nahráním.

## Pravidla adaptace

1. Používej neutrální, smyšlený příklad jako ilustraci a pak ho udělej konkrétní pro firmu a obor uživatele. Nikdy neuváděj reálnou třetí firmu jako vzorový příklad.
2. Nikdy nevymýšlej konkrétní čísla za byznys klienta. Nech prázdná místa, například `_____ Kč`, nebo použij rozsahy. Klient si doplní svá.
3. Drž strukturu metodiky identickou. Mění se jen příklady a název firmy. Neměň délky testů ani logiku OVĚŘENO/NEOVĚŘENO.
4. Dokumenty v jazyce uživatele, česky nebo anglicky. V angličtině přelož obsah věrně, drž stejnou strukturu, pilíře Customer, Problem, Product, Sales, Price, Return a výsledky CONFIRMED a NOT CONFIRMED. Názvy souborů mohou zůstat stejné.
5. Pojmenování složky. Navrhni `RRRR-MM_workshop-kompas/` s podsložkou `_ke-schvaleni/` pro kontrolu a `_finalni/` pro schválené.

## Patička a branding

Každý vygenerovaný dokument zakonči decentní dvouřádkovou patičkou:

```
Kept with Smartin Compass
Work smarter · smartin.work
```

Na titulní dokument (00_rozcestnik) lze přidat i logo Smartin (S v kruhu, Tiffany blue), pokud je asset k dispozici. Jinak stačí patička. Nepřebrandovat, jeden decentní prvek na dokument.

## Doporučení, kterým pilířem začít

- nejasný segment zákazníka → Zákazník
- teoretická bolest → Problém
- klíčová neznámá je cena → Cena
- křehká ekonomika → Návratnost
- nejasná distribuce nebo dodání → Obchod
- slabá diferenciace → Produkt

Výchozí pořadí, když nic není jasně nejrizikovější. Zákazník → Problém → Produkt → Obchod → Cena → Návratnost.
