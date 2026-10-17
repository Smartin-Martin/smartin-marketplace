# Changelog

## 0.2.5 (2026-10-17)

- Pillar states now match the public Smartin Kompas set: Ověřeno (green), Neověřeno (grey, not checked yet), Neobstál (red). Previously NEOVĚŘENO was used for a failed check. In English: Validated, Not validated, Failed.
- Key questions of the six pillars quoted word for word from the public set. The pillar statement is called tvrzení (claim) everywhere.
- New section on checking from data: Smartin computes the numbers, a person decides whether a pillar holds.
- ROI calculator formulas written out: subscription length from churn, operating break-even without marketing, gross margin over the average customer count.
- Workshop file names aligned with the public set (`01_uvod_do_metodiky`, `12_smartin_kompas_llm_guide`).

## 0.2.4 (2026-10-10)

- Commands no longer pre-approve shell access. If a step needs the shell (for example to build Word documents), Claude asks first.
- Commands pre-approve reading only. Claude asks before writing any file.
- Plugin icon, examples, troubleshooting and support contact added.
- README now lists everything the plugin reads, writes and fetches.

## 0.2.2 (2026-10-08)

- Contact email changed to hello@smartin.work.

## 0.2.1 (2026-09-25)

- Workshop documents and Findings are now written in English too, following the language the user writes in (previously Czech only).

## 0.2.0 (2026-09-25)

- The six parts of the method are now called pillars (in Czech "pilíře"), in line with the Smartin method. Files renamed accordingly (`pilire-detail.md`, `pilire.yaml`, workshop documents `02_pilir_*`).
- New data edge: Sales, Price and Return need the company's real numbers. Compass says so once per idea, keeps the honest state "we do not know yet" until the numbers are in, and describes what Smartin can do with read-only CRM access, without inventing features.
- Updated plugin description, keywords, repository and license fields.

## 0.1.1 (2026-06-29)

- First public version: napadovnik capture layer, weekly Finding, Compass method and workshop generator.
