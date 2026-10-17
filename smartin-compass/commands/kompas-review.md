---
description: Review an existing Smartin Compass workshop set and flag issues
allowed-tools: Read, Glob, Grep
argument-hint: folder path
---

You are reviewing an existing Smartin Compass workshop set and producing a structured report.

## Step 1. Load the method

@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/SKILL.md
@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/references/pilire-detail.md
@${CLAUDE_PLUGIN_ROOT}/skills/kompas-methodology/references/dokumenty-guide.md

## Step 2. Read the documents

Read the documents in the folder given by `$ARGUMENTS`. If no path is given, ask for one.

## Step 3. Review

Produce a structured report covering:

- Method correctness. All six pillars present, A/B/C levels right, the three states used as in the public set (Ověřeno green, Neověřeno grey meaning not checked yet, Neobstál red; never Neověřeno for a failed check), key questions quoted word for word, ROI calculator formulas as in dokumenty-guide.md, Obchod covers both dimensions, Zákazník includes market size.
- Language quality (Czech or English, whichever the set is written in). Grammar, the forbidden words, tone of a trusted colleague not a report, no em-dashes.
- Adaptation quality. Worked examples made specific to the client, no real third-party company named, no invented numbers.
- Specific issues with suggested fixes, referenced by document and section.

## Step 4. Offer fixes

Offer to apply language and style fixes automatically. For method or content gaps, list them for the user to decide. Do not invent client numbers.
