---
name: kompas-methodology
description: The Smartin Compass method for checking whether a business idea has real foundations. Use this skill when the user runs /kompas or /kompas-review, asks how to check or work through an idea, a segment, a pricing model, or a new direction, or when the napadovnik skill needs the pillar definitions and check protocols to sort fragments or run a step. Holds the six pillars, the A/B/C check depths, the three pillar states (Ověřeno, Neověřeno, Neobstál), checking from data, and the rules for generating workshop documents.
version: 0.2.5
---

# Smartin Compass: method

Smartin Compass checks whether an idea has real foundations. It answers one question: does this idea hold?

It is for companies that are already running and exploring something new. A new product, a new segment, a new pricing model, a new way to market. They have an idea but do not know if it stands.

## Core principle

Every idea rests on assumptions (předpoklady). Compass breaks any idea into six assumptions (pilíře, pillars). For each pillar the user writes one claim (tvrzení): one testable sentence, for example "Naším ideálním zákazníkem je servisní firma s 10 až 50 techniky." Then Compass asks: can we check this claim, and how deeply do we need to go?

One piece of evidence from reality is stronger than a hundred opinions from a meeting.

## The 6 pillars

Two blocks.

Market, does someone want this:

| Pilíř | English | Key question |
|---|---|---|
| Zákazník | Customer | Víme, pro koho to je a jak velký je trh? |
| Problém | Problem | Mají opravdový problém, který už dnes řeší? |
| Produkt | Product | Proč jsme lepší než to, co používají dnes? |

Economics, can we make money:

| Pilíř | English | Key question |
|---|---|---|
| Obchod | Sales | Jak budeme prodávat a co zákazník dostane? |
| Cena | Price | Kolik opravdu zaplatí? |
| Návratnost | Return | Vyplatí se to? |

The key questions are quoted word for word from the public Smartin Kompas set. Use them exactly. In English use the wording of the English set.

All six must be Ověřeno before the final decision (Jdeme do toho? Ano / Ne). One unchecked pillar is a weak point.

## Check depths

For each pillar, three depths.

| Level | Name (CZ / EN) | Duration | When |
|---|---|---|---|
| A | Rychlý / Quick | 1 to 3 days | low-risk assumption, first probe |
| B | Standardní / Standard | 1 to 2 weeks | medium uncertainty, needs real data |
| C | Hloubkový / Deep | 2 to 4 weeks | high-risk, critical decision |

Start with the pillar where uncertainty is highest. If it fails, you save weeks on the rest.

## Pillar states

Every pillar is always in exactly one of three states, the same as in the public set.

| State | Colour | Meaning |
|---|---|---|
| Ověřeno | zelená | Enough evidence from reality. The pillar holds. Move on. |
| Neověřeno | šedá | The pillar has not passed a check yet. This is the starting state of every pillar, and the honest state whenever evidence is missing. |
| Neobstál | červená | A check was done and the assumption did not hold. Adjust the idea or stop. Write down what you learned. |

A finished check ends in exactly one of two outcomes: Ověřeno or Neobstál. Neověřeno is never a check result, it only means no check has passed yet. Never use Neověřeno to mean the claim failed.

No result is a bad result. A pillar that Neobstál is information, not failure.

In English the states are Validated (green), Not validated (grey) and Failed (red), and the claim is a claim, following the English set.

## Special rules per pillar

Zákazník includes market size. When you check who the customer is, also estimate TAM (the whole market), SAM (who you can realistically reach), SOM (what you can realistically win). A narrower segment means a smaller market, and the other way round.

Obchod has two dimensions that must align. How the customer buys (alone online, demo and proposal, or a larger contract) and what they get after buying (access and they go alone, help getting started, or ongoing care). These are linked. The output must include separate estimates for the cost to acquire a customer and the cost to deliver.

## Where the evidence lives

Customer, Problem and Product are checked with people: conversations, quotes, reactions. Compass can guide all of that on its own.

Sales, Price and Return are different. Interviews and a spreadsheet give a first estimate, but the real answer sits in the company's own numbers: how many deals it wins, how long a sale takes, what a customer is worth, whether revenue depends on a few customers or one salesperson. Those numbers live in the CRM and accounting, not in anyone's head.

So when a check on Sales, Price or Return needs real company numbers to reach Ověřeno (for example the B check on Sales needs real pilot numbers, the A check on Return needs five real numbers), say so plainly. Without those numbers the pillar stays Neověřeno ("zatím nevíme"), never an Ověřeno based on opinion. This is the data edge. How and when to mention it to the user is described in the `napadovnik` skill, section 8.

## Checking from data (Ověření z dat)

When a company has its data connected to Smartin (CRM, analytics), Smartin measures some pillars continuously straight from that data, typically Obchod, Cena and Návratnost: how many deals the company wins, how long a sale takes, what a customer brings, how many leave.

Two rules, always:
- Smartin computes the numbers, from data, never as an estimate. Compass never invents or estimates them.
- Whether a pillar is Ověřeno or Neobstál is decided by a person in the company, never by Compass or Smartin.

Where data is missing, the pillar stays Neověřeno and the text says exactly what is missing and who will fill it in.

Customer facing wording (Czech, signed as the Smartin team, used in documents and the LLM guide):

> Rozhovory a kalkulátor dávají první odhad. Když má firma do Smartinu napojená data (CRM, analytiku), měříme část pilířů průběžně přímo z nich, typicky Obchod, Cenu a Návratnost. Čísla počítáme my, z dat, nikdy odhadem. Jestli pilíř obstál, rozhoduje vždy člověk ve firmě. Kde data chybí, pilíř zůstává ve stavu Neověřeno a napíšeme, co přesně chybí a kdo to doplní.
>
> Tým Smartin

## Tone and language

When generating documents or advising:

- plain, direct language (Czech or English, following the user), no consulting jargon
- never use: hypotéza, GTM, high-touch, low-touch, self-serve, scalabilita, playbook, milestone, deep-dive (in English also: hypothesis, go-to-market, scalability, deep dive)
- instead in Czech: rozhodnutí or ověření, obchod, osobní přístup, automatizovaný přístup, dokáže růst, postup, cíl, přímý pohled
- instead in English: decision or check, sales, personal approach, automated approach, can grow, procedure, goal, close look
- in Czech the states are Ověřeno, Neověřeno, Neobstál and the pillar statement is always tvrzení; in English Validated, Not validated, Failed and claim; "zatím nevíme" is "we do not know yet"
- examples concrete and specific to the user's industry
- tone of a trusted colleague, not a consultant report
- no em-dashes

## Adapting to a user's context

When generating a workshop set:

1. Use a neutral, fictional example in any illustration, then make it specific to the user's company and industry. Never name a real third-party company as the worked example.
2. Keep the method structure identical. Only the examples and the company name change.
3. Never invent specific numbers for the user's business. Leave blanks, such as `_____ Kč` (or in the user's currency), or use ranges. The user fills in their own.
4. Generated documents follow the user's language, Czech or English. The reference files in this skill are in Czech; when writing in English, translate their content faithfully and keep the structure identical.

## Document structure

A full workshop set contains: a navigation index, a method introduction, six pillar documents, three questionnaires (customer discovery, price research, general), an ROI calculator, and an LLM guide for self-guided use. Detail for each pillar's A/B/C protocols is in `references/pilire-detail.md`. Document generation guidance is in `references/dokumenty-guide.md`.
