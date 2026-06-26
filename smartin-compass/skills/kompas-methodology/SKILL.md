---
name: kompas-methodology
description: The Smartin Compass method for checking whether a business idea has real foundations. Use this skill when the user runs /kompas or /kompas-review, asks how to check or work through an idea, a segment, a pricing model, or a new direction, or when the napadovnik skill needs the tile definitions and check protocols to sort fragments or run a step. Holds the six tiles, the A/B/C check depths, the OVĚŘENO and NEOVĚŘENO logic, and the rules for generating workshop documents.
version: 0.1.0
---

# Smartin Compass — method

Smartin Compass checks whether an idea has real foundations. It answers one question: does this idea hold?

It is for companies that are already running and exploring something new. A new product, a new segment, a new pricing model, a new way to market. They have an idea but do not know if it stands.

## Core principle

Every idea rests on assumptions. Compass breaks any idea into six assumptions (kostičky, tiles) and for each asks: can we check this, and how deeply do we need to go?

One piece of evidence from reality is stronger than a hundred opinions from a meeting.

## The 6 tiles

Two blocks.

Market, does someone want this:

| Tile | Czech | Key question |
|---|---|---|
| Zákazník | Customer | Do we know who this is for? |
| Problém | Problem | Do they have a real problem? |
| Produkt | Product | Why are we better than the alternatives? |

Economics, can we make money:

| Tile | Czech | Key question |
|---|---|---|
| Obchod | Sales and delivery | How do we sell and what do we deliver? |
| Cena | Price | How much will they pay? |
| Návratnost | Return | Does it pay off? |

All six must hold before an idea is ready to run. One unchecked tile is a weak point.

## Check depths

For each tile, three depths.

| Level | Czech | Duration | When |
|---|---|---|---|
| A | Rychlý | 1 to 3 days | low-risk assumption, first probe |
| B | Standardní | 1 to 2 weeks | medium uncertainty, needs real data |
| C | Hloubkový | 2 to 4 weeks | high-risk, critical decision |

Start with the tile where uncertainty is highest. If it fails, you save weeks on the rest.

## Result logic

After any check there is exactly one outcome.

- OVĚŘENO. Enough evidence. The tile holds. Move on.
- NEOVĚŘENO. Not confirmed. Adjust the idea or stop. Write down what you learned.

No result is a bad result. An unchecked tile that turns out NEOVĚŘENO is information, not failure.

## Special rules per tile

Zákazník includes market size. When you check who the customer is, also estimate TAM (the whole market), SAM (who you can realistically reach), SOM (what you can realistically win). A narrower segment means a smaller market, and the other way round.

Obchod has two dimensions that must align. How the customer buys (alone online, demo and proposal, or a larger contract) and what they get after buying (access and they go alone, help getting started, or ongoing care). These are linked. The output must include separate estimates for the cost to acquire a customer and the cost to deliver.

## Tone and language

When generating documents or advising:

- plain, direct Czech, no consulting jargon
- never use: hypotéza, GTM, high-touch, low-touch, self-serve, scalabilita, playbook, milestone, deep-dive
- instead: rozhodnutí or ověření, obchod, osobní přístup, automatizovaný přístup, dokáže růst, postup, cíl, přímý pohled
- examples concrete and specific to the user's industry
- tone of a trusted colleague, not a consultant report
- no em-dashes

## Adapting to a user's context

When generating a workshop set:

1. Use a neutral, fictional example in any illustration, then make it specific to the user's company and industry. Never name a real third-party company as the worked example.
2. Keep the method structure identical. Only the examples and the company name change.
3. Never invent specific numbers for the user's business. Leave blanks, such as `_____ Kč`, or use ranges. The user fills in their own.
4. All generated documents are in Czech by default. The LLM guide may include other languages on request.

## Document structure

A full workshop set contains: a navigation index, a method introduction, six tile documents, three questionnaires (customer discovery, price research, general), an ROI calculator, and an LLM guide for self-guided use. Detail for each tile's A/B/C protocols is in `references/kosticky-detail.md`. Document generation guidance is in `references/dokumenty-guide.md`.
