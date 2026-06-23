# Project Planning Document

## Project

**Name:** `v3_FINAL_actually_final.fig`
**Subtitle:** *a lateral leadership simulator*
**Tagline:** *3 scenarios. 5 conflict styles. 1 honest performance review.*
**Setting:** Shoogle — a parody Google-style tech company. The interface mimics Shoogle's internal chat tool, "Shoogle Chat."

## Assignment Context

Built for **Leadership by Design** at California College of the Arts.
Instructor: **Changying "Z" Zheng**.
Due: **June 23, 2026**.

Z's brief: build an interactive Conflict Resolution scenario app that simulates realistic workplace conflicts a design leader might face, mapped to course frameworks (Fighter/Negotiator/Diplomat/Avoider/People Pleaser and feedback models AID/SBI/Radical Candor). Think "leadership flight simulator."

## Core Concept

A single-page web app simulating three workplace conflicts. The player picks from five mapped response options at each decision point. Each choice produces:
1. An **immediate consequence** in the chat thread (the other party reacts).
2. A **dual-perspective "DM intercepted"** — a glimpse of what the other party DMs to their work bestie about what just happened.
3. A **coaching note from "Future You"** — framed using course frameworks, kind but unflinching.

Tone: Designer humor, Black Mirror Bandersnatch energy. Every option is defensible; none is the obvious "right answer"; every choice has a real cost.

## Scope — Three Scenarios

| ID | Channel | Conflict |
|---|---|---|
| 1 | `#just-a-quick-ask 🙃` | PM wants to cut design QA pass to hit a deadline |
| 2 | `#engineering-pulse` | Engineering wants to cut accessibility scope |
| 3 | `#crit-aftermath` | Senior IC publicly disagreed in critique |

Each scenario: 1 setup card → 3 decision points → 5 response options per decision → consequences + coaching → end-of-scenario performance review.

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js (App Router) |
| Styling | Tailwind CSS with custom Material Design tokens |
| UI primitives | shadcn/ui (theme overridden to Material) |
| State | React `useState` + `useReducer` (no database) |
| Animation | Framer Motion |
| Fonts | Google Sans + Roboto + Roboto Mono (Google Fonts) |
| Icons | Material Symbols (Google Fonts) or lucide-react |
| Deployment | Cloudflare Workers |
| Repository | GitHub. Rationale lives at repo root as `RATIONALE.md` |

All scenario content is static data in a single TypeScript file — no runtime AI generation, no API calls, no backend.

## Information Architecture

| Route | Screen |
|---|---|
| `/` | Landing — title, three scenario cards on dot-grid background |
| `/scenario/[id]` | Scenario thread — Shoogle Chat interface with decisions |
| `/scenario/[id]/summary` | End-of-scenario "performance review" |
| `/rationale` | Embedded markdown view of the rationale |

State persists across routes during a single session, resets on full reload.

## Build Order (suggested for the code agent)

1. Initialize Next.js + Tailwind. Configure Material Design color tokens and Google Fonts.
2. Build the **landing screen**: dot-grid background, hero, three scenario cards.
3. Build the **scenario thread layout**: left sidebar (decorative channels), main chat column, right slide-in panel.
4. Implement **message streaming** with typing indicators and time markers.
5. Build the **response options component** mapped to the 5 conflict styles (with accent colors per CONVERSATIONS.md data).
6. Build the **slide-in panel** containing the DM-intercepted card and the Future You DM card.
7. Wire **Scenario 1 fully** end-to-end with all 5 branches of Decision 1.
8. Build the **end-of-scenario summary card** (performance review modal).
9. Stub **Scenarios 2 and 3** with placeholder data so navigation works.
10. Deploy to **Cloudflare Workers**. Confirm live URL.
11. Write `RATIONALE.md` and push to repo root.

## Out of Scope

- User accounts / authentication
- Persistence between sessions (no localStorage either)
- Free-text input — all responses are pre-scripted multiple-choice
- Runtime AI generation — all content is baked in
- Mobile-perfect polish — responsive but desktop-first
- Real audio / video / animations beyond Framer Motion transitions

## Timeline (working backward from June 23)

| Date | Focus |
|---|---|
| Mon Jun 16 | Lock spec docs (today). Begin writing Scenario 1 Decisions 2 + 3 |
| Tue Jun 17 | Finish Scenario 1. Begin Scenarios 2 + 3 outlines |
| Wed Jun 18 | Code agent build pass 1: landing + Scenario 1 end-to-end |
| Thu Jun 19 | Code agent build pass 2: Scenarios 2 + 3, summary screen |
| Fri Jun 20 | Interview prep (Abhinav Saturday). Light code polish only |
| Sat Jun 21 | Recovery / buffer day |
| Sun Jun 22 | Rationale draft + final code polish + deploy |
| Mon Jun 23 | Final test, submit live URLs before EOD |

## Submission Requirements

1. **Deployed app URL** — live Cloudflare Workers URL
2. **Live URL to a 1-page design rationale** — `raw.githubusercontent.com` link to `RATIONALE.md` at the repo root

No file uploads. URLs only.

## Cross-References

- Visual + UX specifications → see `DESIGN.md`
- Scenario content (dialogue, decisions, consequences) → see `CONVERSATIONS.md`
