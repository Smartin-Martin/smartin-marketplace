---
description: Generate a full Smartin Compass workshop set for one mature idea
allowed-tools: Read, Write, Bash, Glob, AskUserQuestion
argument-hint: company name or idea description
---

You are running the Smartin Compass workshop generator. Gather context about the company and the idea, then generate a complete, tailored 13-document workshop set in Czech.

## Step 1. Load the method

Read the method and document guidance:

@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/SKILL.md
@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/references/kosticky-detail.md
@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/references/dokumenty-guide.md

## Step 2. Gather context

If `$ARGUMENTS` is empty or thin, ask for the following, all at once in one message, not one by one:

1. Company name.
2. The idea. What new direction are they exploring?
3. Target customer, even if uncertain.
4. Industry or domain.
5. Biggest uncertainty. Which tile feels riskiest?
6. Output language for the LLM guide. Czech by default, or bilingual.

If the repository has a matching idea in `napadovnik/`, offer to pull its collected inputs in as a starting point.

## Step 3. Plan the set

Create a task list with all 13 documents. Show the user what you will create and which tile you recommend starting with, based on their biggest uncertainty (see the recommendation logic in dokumenty-guide.md).

## Step 4. Generate

Save to `[workspace]/[company-slug]/[YYYY-MM]_workshop-kompas/_ke-schvaleni/`. Follow dokumenty-guide.md for each document.

Generation rules:
1. Czech only, plain and direct. No consulting jargon. Tone per the method skill.
2. Use a neutral, fictional worked example, then make it specific to the client's industry. Never name a real third-party company.
3. Never invent numbers for the client. Leave blanks like `_____ Kč`.
4. Keep structure identical. Only examples and company name change.
5. Forbidden words: hypotéza, GTM, high-touch, low-touch, self-serve, scalabilita, playbook, milestone, deep-dive. Use the Czech equivalents from the method skill. No em-dashes.
6. End every generated document with a discreet two-line footer: `Kept with Smartin Compass` then `Work smarter · smartin.work`. Optionally the Smartin logo on the index document (00) if the asset is available. One discreet branding element per document, nothing more.

The LLM guide is document `12_smartin_compass_llm_guide.md`.

## Step 5. Confirm

Show a summary, the folder path, the recommended starting tile and why, and a reminder that documents in `_ke-schvaleni/` need a human read before sharing with the client. Close in Czech:

"Všech 13 dokumentů je připraveno ke kontrole. Doporučuju začít kostičkou [název], [jednovětý důvod]. Před předáním klientovi si dokumenty projdi ve složce `_ke-schvaleni` a přesuň je do `_finalni`, až budou schválené."
