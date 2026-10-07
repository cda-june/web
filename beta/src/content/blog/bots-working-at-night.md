---
title: "Agents That Work While You Sleep: Claude Code as an Autonomous Daemon"
description: "Autonomous agents watching a team chat, implementing code, opening pull requests, and rolling out hotfixes, and the lessons that explain every single guardrail."
excerpt: "A word in the chat wakes an agent. It implements, opens a PR, reports back. Several personas from one building kit. And the night an agent burned a lot of tokens in an idle loop."
category: "Autonomy"
image: "/images/blog/bots-die-nachts-arbeiten.svg"
order: 5
date: 2026-07-22
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "10 min"
published: false
lang: "EN"
---

This is the article the client question quoted in the first part was really aiming at: *"You type in feature requests as text, and then agents go off, implement them, open pull requests?"*

Yes. That's how it works. And this is how we built it at JUNE.

## The basic idea: an agent is Claude Code as a daemon

An "agent" is nothing more than **Claude Code running as a long-lived daemon in a container**, driven by chat messages instead of a human at a terminal. It watches a team chat channel, and as soon as a trigger word appears, it implements code changes, opens pull requests, addresses review comments, rolls out hotfixes, or tests the application in the browser, all unattended.

The key is in the architecture: there are several personas (a developer agent, a support agent, a product management agent, a test agent), but they are **not separate codebases.** They're the same runtime, specialized only through a different system prompt, a different list of installed skills, and a few environment variables.

> "A new agent is just a container with different environment variables and a different system prompt."

That's the DRY principle at the agent level. An improvement to the shared building kit reaches them all immediately.

## The trigger: a simple poll, not a webhook

You'd expect such a system to be driven by webhooks. It isn't. Each agent is a **short-interval poll loop.** Over and over, it checks the chat: is there a new message with the trigger word? If so, it starts the language model. If not, it keeps sleeping, without burning a single token. No webhook registration that silently breaks, no externally exposed interface. And to hide the perceived latency, there's a neat UX trick: even before the model starts, the poll posts a pre-confirmation, "I'm on it! 🐳", that keeps updating with the current work step. The human sees a reaction right away instead of waiting for the first token.

## How it runs Claude Code headless

At its core, the bootstrap calls Claude Code in **headless mode**, with permission prompts skipped. The agent isn't supposed to ask about every file. That's exactly why one of the most important guardrails is a **hard stop via hook**: a merge to `master` or `main` is categorically refused.

> "Autonomous merging is disabled … leave the merge to a human."

The agent may push, may open pull requests, but merging into the main line remains a human decision. That's the "hand on the brake lever" the entire system is guided by. Because permissions are skipped, this stop must be a *hard* code stop. A mere prompt rule would just be advice the model might ignore in the heat of the moment.

## The lessons

Nearly every guardrail in this system traces back to a concrete experience in day-to-day operation. That isn't embarrassing; it's the method: **the system grows by pouring its own errors into code.**

**The night in an idle loop.** An agent's chat token had expired. The poll interpreted this as "there's work" and fired the language model again and again to "solve" the supposed problem, all night long. In the morning: a lot of token cost for nothing. The answer was *several* independent cost guards: a silent token refresh that first tries to solve the problem without the model; an error state machine that throttles hard after repeated failures; and a weekly limit marker. Since then, an expired token *never* fires the model; the poll simply skips the tick.

The cost guards stopped the burning, but they brought their own failure mode: a genuinely stuck agent now retries only rarely. If it's really broken, you find out late.

**The configuration on the network drive.** Initially, the agent configuration lived on a persistent network drive. There, cloning and resetting the Git repository kept failing and left corrupted files behind. The broken folder couldn't be deleted, and the agent got stuck. The lesson: configuration belongs on volatile local storage, freshly cloned on each start; only the *state* lives persistently. And never `sleep infinity` on failure; better to exit cleanly and let the container start a fresh process.

**The self-restart loop.** The agents learn: after a review comment, they write a new rule to their knowledge base and push it. Initially, the deployment automation interpreted this push as a configuration change, and restarted the agent mid-work. After the restart, the agent repeated the work, learned, committed, pushed, restarted … an infinite self-restart loop. The fix: explicitly exclude knowledge-base pushes from the restart logic.

**"Never rely on sender identity."** Because the agent posts via a human's token, agent and human share a display name. A case where the agent reacted to its own status message, because it contained the trigger word, led to the rule: everything keys off message IDs, never to the display identity.

**"Done only counts as learned when it's written down."** The test agent, which clicks through the application in the browser, maintains its own knowledge base about the product's surface. The guiding principle behind it: experience that isn't recorded anywhere is lost. So the agents write down their lessons and push them, deploy-neutral, immediately available to all.

## The self-learning pillar

These agents get better over the months instead of staying equally bad: after every review, every correction, they write generalizable rules to a knowledge base and share them. The developer agent learns frontend conventions, the support agent learns the choreography of a rollout, the test agent learns the quirks of the surface. **The tools improve the documents that control the tools**, the same compounding pattern as in the marketplace.

## What to take from this

Autonomous agents in production aren't a magic trick. They're a very ordinary tool (Claude Code) in a very disciplined environment: a cheap poll instead of fragile webhooks, hard code boundaries around risky actions, several independent cost guards, and a culture where every experience becomes a new rule.

The hardest part isn't getting the agent to work. The hardest part is giving it the boundaries that let you sleep at night.

In the final part of the series, I turn the perspective around: away from the machines working autonomously, toward a single human and the cockpit that orchestrates their day with many agents.
