---
title: "The Nervous System: Event Automation and How to Keep a Language Model Honest"
description: "How an always-on automation pillar turns support emails into classified tickets and a multi-stage privacy filter that shows what 'don't trust the model, verify with code' looks like in practice."
excerpt: "Many workflows, a lot of processing steps, no human in the loop. A self-learning ticket router and a privacy filter that lets the model write, but doesn't believe a single word."
category: "Automation"
image: "/images/blog/nervensystem-n8n-automatisierung.svg"
order: 3
date: 2026-07-15
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "9 min"
published: false
lang: "EN"
---

If the foundations, tickets and chat, are the skeleton of the system, then the automation pillar is the nervous system: always awake, event-driven, no human in the loop. It reacts to every change in a ticket and controls the other systems from there.

At JUNE, we built this pillar with n8n, an open-source automation platform. Many workflows, a lot of processing steps. It turns a support email into a classified ticket, a call recording into a structured task list, a comment into finished release notes. Two of these workflows deserve a closer look because they rest on two principles that explain the rest of the stack.

## First: code is the truth, not manual work

The defining decision of this pillar: for every non-trivial workflow, the exported configuration is the source of truth, but it isn't written by hand. A small Python script *generates* it.

Why? Because hand-editing large workflow definitions keeps producing the same errors: wrong node IDs, broken connection arrays, type mix-ups. A generator script doesn't make these mistakes. Both the generator and the generated configuration live in Git. It's the same philosophy running through the entire stack: **where something can be deterministic, it should be a script.**

## Second: a router that learns from human corrections

The support workflow is a self-improving loop of two parts.

The first part catches every new support conversation and creates a ticket from it. Then a language model classifies the ticket: which list does it belong to? The classification runs as a **few-shot prompt**: the model gets examples of past tickets with the list they were sorted into. If it's confident enough, it moves the ticket automatically. If it's unsure, the ticket stays in the inbox.

The second part closes the loop: whenever a *human* manually moves a ticket from the inbox, that exact correction is saved as a new example. The router's training data *is* the log of human corrections. There's no separate labeling step. On day one, with an empty example table, the system simply skips the model and leaves everything in the inbox, and learns from the first manual move onward.

**The corrections your team already makes every day are the training data.** You just have to capture them.

One value in there is honest guesswork: the confidence threshold above which the model may move a ticket itself. I picked it by feel, not by tuning. It's held up so far. Whether it's set right or I've just been lucky, I still don't know.

And the whole thing is designed to be fault-tolerant at every branch: if classification fails, the ticket already sits in the inbox, which is a safe fallback. Nothing is lost just because the model makes a mistake.

## The prime example: the privacy filter

When someone asks me how to make a language model safe in production, I show them this workflow.

JUNE generates customer-facing release notes automatically from internal tickets. Internal tickets are written for colleagues, not for the public: they hold details that have no place in a release note. So from the start there is a deterministic boundary between ticket and note, not merely an instruction in the prompt.

The naive solution would be: tell the model in the prompt "don't mention names". I do that too: the prompt contains a hard prohibition with examples. **But I don't trust the prompt.** So the flow runs in stages:

1. **The model writes** the release note, with instructions to generalize everything.
2. **A deterministic filter checks** the result: regex searches for emails, URLs, phone numbers, IDs. Plus a heuristic that flags capitalized words that aren't at the start of a sentence and aren't on a small allowlist as likely names.
3. **If the filter fires, a second model redacts** the flagged spots: it is told to remove or generalize every flagged term.
4. **The same filter runs a second time.**
5. **If it *still* triggers, the workflow refuses hard** and posts a comment instead: "Sanitizer could not remove sensitive content, please rewrite manually."

**Model writes → code checks → model corrects → code checks again → hard refusal if in doubt.** The language model is used where it shines (fluent formulation, generalizing), but the fenced area is narrow, and the boundary is deterministic code, not another model.

The filter code is deliberately duplicated verbatim in two places (instead of being moved into a shared function), with the comment "keep this block in sync with the other one". Sometimes redundancy is the right decision.

## The thread

The router gets smarter from human corrections; the filter doesn't believe a single word the model writes. Those two patterns carry the whole pillar. That's why it runs all night without a human in the loop.

In the next part, I look at the pillar above: the marketplace that packages company knowledge into installable skills, the "apps" that both humans *and* agents use.
