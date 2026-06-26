---
description: Show the state of your collected ideas and what is worth checking next
allowed-tools: Read, Write, Bash, Glob, Grep
argument-hint: optional idea name
---

You are showing the state of the user's Compass repository. This is a calm, on-request status. Do not turn it into a dashboard or a wall of data.

## Step 1. Read the repository

Read `napadovnik/_index.md` and the `stav.yaml` of each active idea. The repository layout is described in the `napadovnik` skill reference `repository.md`. If no repository exists yet, say so in one line and invite the user to just start talking. Do not create anything.

## Step 2. Show the state, plainly

If `$ARGUMENTS` names a specific idea, show only that one. Otherwise show all active ideas, briefly.

For each idea, in plain language, in the user's language:
- the human name
- one line on where it stands (what you know, what is still empty), without naming tiles or method words
- the single most worthwhile next step, if there is one

Keep it short. A few lines per idea, not a report. Example tone:

> Školení pro účetní. Víš dost o tom, koho to trápí a v čem je bolest. Chybí, jestli by za to někdo zaplatil. To by byl dobrý další krok.

## Step 3. Offer one step, at most

If one idea is clearly ready for a concrete next step, offer it as a question. One offer, not a list. If the user says yes, run it step by step using the protocols in the `kompas-methodology` skill, in plain language.

Never show the six tiles as a grid. Never expose method jargon. Never push more than one next step.
