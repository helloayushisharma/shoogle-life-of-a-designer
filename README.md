# Life of a designer

*A lateral leadership simulator.*

**3 scenarios. 5 conflict styles. 1 honest performance review.**

**▶ Play it live → https://helloayushisharma.github.io/shoogle-life-of-a-designer/**

No login, no build, no install — it opens and plays in any browser.

---

## What it is

You play a design lead at **Shoogle** — a parody of a Google-style company — navigating realistic
workplace conflicts inside **Shoogle Chat**, the company's internal messaging tool. A PM wants to
cut design QA to hit a deadline. Engineering wants to drop accessibility scope. A senior IC
publicly disagrees with your direction in a critique.

At every decision point you respond in one of **five conflict styles**, and each choice produces:

- an **immediate in-thread consequence** — the other party reacts, in channel;
- an **intercepted DM** — a glimpse of what they're saying about you behind your back;
- a **coaching note from "Future You"** — kind but unflinching, framed with real leadership models.

Every option is defensible. There is no single "right" answer, and every choice has a real cost.
Each scenario ends with a **performance review** that names your dominant conflict style and tells
you what it cost you and what it earned you.

## How to play

1. Open the [live demo](https://helloayushisharma.github.io/shoogle-life-of-a-designer/).
2. Pick a scenario from the hub — play them in any order.
3. Read the thread as it unfolds, then choose one of the five response options.
4. Watch the consequence land, read the intercepted DM and the note from Future You, and continue.
5. Make it through all three decisions to get your performance review.

State lives in memory only — a full page reload resets everything.

> **Dev shortcut:** press **`D`** (or click the **⚙ dev pill** at the bottom-left) to jump
> straight to any scenario, decision, or summary.

## The three scenarios

| Channel                  | Conflict                                                  | Difficulty   |
| ------------------------ | --------------------------------------------------------- | ------------ |
| `#just-a-quick-ask 🙃`   | A PM wants to cut the design QA pass to hit a deadline    | 🔥 Familiar  |
| `#engineering-pulse 📈`  | Engineering wants to cut accessibility scope to ship      | 🔥🔥 Tense   |
| `#crit-aftermath`        | A senior IC publicly disagreed with your direction        | 💀 Personal  |

Each scenario has **3 decisions** and ends with its own performance-review summary.

## The five conflict styles

Every decision offers the same five mapped responses:

- **Fighter** — hold the line, challenge directly.
- **Negotiator** — trade, compromise, find the deal.
- **Diplomat** — reframe the problem, collaborate toward what everyone actually needs.
- **Avoider** — defer, buy time, step back from the conflict.
- **People-Pleaser** — accommodate, smooth it over, take the path of least resistance.

## Frameworks behind the coaching

Future You's notes are grounded in established leadership and feedback models:

- **Thomas-Kilmann** conflict modes (Compete / Collaborate / Compromise / Avoid / Accommodate)
- **Radical Candor** — challenge directly while caring personally
- **AID** — Action · Impact · Do-differently
- **SBI** — Situation · Behavior · Impact

## Run locally

The app is a single self-contained `index.html` — no build step, no dependencies.

- **Simplest:** open `index.html` directly in any browser.
- **Or serve it** (handy if your browser is strict about local files):

  ```bash
  python3 -m http.server 8000
  # then visit http://localhost:8000
  ```

## Project structure

| File                 | What it is                                                          |
| -------------------- | ------------------------------------------------------------------- |
| `index.html`         | The entire app — HTML, CSS, and vanilla JS in one file.             |
| `PLANNING.md`        | Architecture and scope spec.                                        |
| `DESIGN.md`          | Visual identity, layout, and interaction spec.                      |
| `CONVERSATIONS.md`   | All scenario content — dialogue, decisions, consequences, coaching. |

Styling follows **Google Material Design**, with **Google Fonts** loaded via CDN.

## Credits

Built by **Ayushi Sharma** for **Leadership by Design** at the **California College of the Arts (CCA)**,
instructor Changying "Z" Zheng.
