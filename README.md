# Kórtex Mission Control

An operations panel for clinics running AI phone and messaging agents.

Kórtex installs and operates AI agents for Spanish healthcare centers (radiology and diagnostic imaging, radiotherapy, dental, aesthetic medicine, nursing homes). Each agent has a name and one job: answering the phone, booking and moving appointments, chasing no-shows, replying on WhatsApp and email, and handling Google reviews.

Mission Control is the SaaS panel where clinic staff see and steer what their agents do. This repo documents its design and current state of development.

**Status: in active development, roughly 63% built.** The design phase is done (23 screens, all in this repo). The panel application itself is being built on top of it; the agents already run in service mode without the self-service panel.

---

## What the panel does

![Mission Control home: live agent activity and recovered revenue](screenshots/panel-inicio.webp)

A clinic manager opens Mission Control in the morning and sees the live feed: each call, booking, reminder and recovered slot, with a timestamp and which agent did it. Next to it, the numbers that matter to a manager: how much revenue the agents recovered this month (with the formula visible and the average appointment value editable, because clinics rightly distrust magic numbers) and how many voice minutes and WhatsApp conversations they have used, with spending caps.

The screen I care most about is the review queue. Anything delicate, like a payment dispute or an upset patient, never gets decided by an agent. It waits for a human with the full context attached, and a person can take over the conversation from that same screen. Every action in the system is logged and exportable, and agents always introduce themselves as AI assistants.

### More screens

| Review queue | Conversation detail |
|---|---|
| ![Review queue](screenshots/panel-cola.webp) | ![Conversation with transcript](screenshots/panel-conversacion.webp) |

| Calendar | Weekly report |
|---|---|
| ![Agenda view](screenshots/panel-agenda.webp) | ![Weekly WhatsApp report](screenshots/panel-informe.webp) |

The full set of 23 high-fidelity screens lives in [`screens/`](screens/) as self-contained HTML on top of the Kórtex design system (`kortex-ds.css`): access, home, review queue, conversations, team, onboarding, calendar, integrations, consumption, compliance, agent configuration, clinic settings, users and roles, notifications, system states, first-day experience, plus vertical-specific homes for dental clinics, nursing homes and diagnostic imaging.

## Agent architecture

Each agent is an independent, single-mission worker orchestrated around the clinic's existing tools. No rip-and-replace: agents work on top of the calendar, phone number and WhatsApp line the clinic already has.

| Agent | Mission | Core stack |
|---|---|---|
| Voice | Answers calls 24/7, resolves routine matters, books appointments, warm-transfers anything delicate | Telnyx Voice AI (EU-resident stack) + neural TTS |
| Calendar | Creates, moves and cancels appointments on the clinic's real calendar, with clinic rules (rooms, equipment, timings) | Google Calendar / Microsoft Graph / Cal.com, orchestrated with n8n |
| Reminders | Confirms appointments, rescues no-shows, refills freed slots from the waiting list | WhatsApp Business Cloud API (utility templates) + SMS fallback |
| WhatsApp | Handles the clinic's WhatsApp line: logistics, directions, preparation instructions, new bookings | WhatsApp Business Cloud API + LLM |
| Email | Triages the front-desk inbox, answers the routine, drafts the rest for human review | IMAP / Microsoft Graph + LLM |
| Reviews | Replies to Google reviews through a human-approved queue, requests reviews after visits | Google Business Profile API |

Two design decisions shape everything. First: humans decide. Agents execute the repetitive work and escalate anything clinical or delicate to staff; there is no autonomous clinical decision anywhere in the system. Second: EU data residency. Voice, inference and storage run on providers with EU-resident infrastructure under data processing agreements, and agents handle scheduling and messaging only; they don't access medical records. Traceability and EU AI Act transparency duties are treated as product surfaces the clinic can show an auditor.

The same engine serves dental clinics, imaging centers, radiotherapy departments and nursing homes: one codebase, configured per vertical.

## Technology

The LLM layer uses structured outputs with strict per-agent action allowlists, on EU-resident inference endpoints for anything patient-adjacent. Voice runs on Telnyx (carrier, GPUs and storage in Europe) with neural TTS tuned for natural Castilian Spanish. Messaging goes through the WhatsApp Business Cloud API with utility templates only, plus SMS fallback via Spanish routes. n8n is the workflow backbone connecting voice, calendars, messaging and the panel. The front end is plain semantic HTML on a token-based design system, responsive from 360px up.

## What's done and what isn't

- [x] Product design: 23 high-fidelity screens across 4 verticals
- [x] Design system with brand tokens and an accessibility pass
- [x] Agent service architecture and EU-resident provider selection
- [ ] Panel application build (in progress; the build is behind the design, which is normal and fine)
- [ ] Live pilot with founding clinics
- [ ] Self-service onboarding

## About

Kórtex is built by [VaultBit](https://github.com/vaultbit-web), a Spanish software engineering studio focused on AI agents and security. Website: [kortexagents.com](https://www.kortexagents.com).

All rights reserved. This repository is a portfolio showcase; the design system, screens and brand assets are proprietary and may not be reused without written permission.
