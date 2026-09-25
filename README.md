# Smartin Compass: a free Claude Code and Claude Cowork plugin

Smartin Compass helps a running company check whether a new idea actually holds before betting time on it. A new product, a new segment, a new pricing model, a new way to sell. You capture raw ideas as they come, see which ones are worth checking, and turn the week into one clean Finding worth forwarding. It is free and local-first, so your ideas and evidence stay in your own files.

It is built on the Smartin Compass method: one piece of evidence from reality beats a hundred opinions from a meeting.

## Install

```
/plugin marketplace add Smartin-Martin/smartin-marketplace
/plugin install smartin-compass
```

Reload plugins if prompted with `/reload-plugins`.

In Claude Cowork, add the same marketplace (`Smartin-Martin/smartin-marketplace`) in the plugin settings and install Smartin Compass from there.

## The method: six pillars

Every idea rests on assumptions. Compass breaks any idea into six pillars, a simple canvas, and checks each one against reality. All six must hold before an idea is ready to run. One unchecked pillar is a weak point.

Market, does someone want this:

| Pillar | Key question |
| --- | --- |
| Customer | Do we know who this is for, and how big the market is? |
| Problem | Do they have a real problem they already try to solve? |
| Product | Why are we clearly better than the alternatives? |

Economics, can we make money:

| Pillar | Key question |
| --- | --- |
| Sales | How do we sell, what do we deliver, and what does it cost? |
| Price | How much will they actually pay? |
| Return | Do the unit economics pay off? |

## How a check works

For each pillar you choose a depth based on risk. A quick check (1 to 3 days) is a first probe. A standard check (1 to 2 weeks) brings real data from customer conversations. A deep check (2 to 4 weeks) is for high-risk, critical decisions. Start with the pillar where uncertainty is highest. If it fails, you save weeks on the rest.

Every check ends in one of two outcomes. Confirmed: enough evidence, the pillar holds, move on. Not confirmed: adjust the idea or stop, and write down what you learned. An assumption that turns out not to hold is information, not failure.

## When a pillar needs your real numbers

Customer, Problem and Product are checked with people: conversations, quotes, reactions. Compass guides all of that on its own.

Sales, Price and Return are decided by numbers the company already has in its CRM and accounting: how many deals it wins, how long a sale takes, what a customer brings over time. Until those numbers are in, the honest answer for these pillars is "we do not know yet", not a guess dressed up as a result.

When an idea reaches that point, Compass says so once and leaves the decision to you. If you want to go further, Smartin connects to your CRM and analytics with read-only access and checks those pillars against your real data every week. That part is paid. If you say no, Compass keeps working with interviews and estimates.

## Commands

The plugin adds four commands. `/compass` shows the state of your collected ideas and what is worth checking next. `/finding` produces this week's Finding, one clean page worth forwarding. `/kompas` generates a full workshop set for one mature idea: a method intro, six pillar documents, customer and pricing questionnaires, and an ROI calculator. `/kompas-review` reviews an existing workshop set and flags issues.

## Skills

Two skills load automatically. The napadovnik skill catches half-thoughts, links, and customer quotes from conversation and quietly matures them, with zero friction. The kompas-methodology skill holds the six pillars, the check depths, and the rules for turning an idea into a workshop you can actually run.

## Why it matters

Most ideas die from an assumption nobody checked. Compass makes the assumptions visible, tells you which one to check first, and keeps the cost of being wrong measured in days instead of quarters.

## About

Smartin Compass is the free front door of Smartin, a decision system for company leadership. Smartin is in private beta. Learn more at https://smartin.work.

Keywords: Claude Code plugin, Claude Cowork plugin, Claude plugin marketplace, Anthropic, MCP, Agent Skills, idea validation, business model canvas, decision-making, unit economics, customer discovery, pricing research, Smartin Compass.
