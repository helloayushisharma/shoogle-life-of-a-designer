# Scenario Content & Data Structure

This document contains the actual game content: setup cards, dialogues, response options, consequences, and the TypeScript data structure the code agent should use. Use alongside `PLANNING.md` (architecture) and `DESIGN.md` (visual specs).

---

## Status

| Scenario | Decision 1 | Decision 2 | Decision 3 | Summary |
|---|---|---|---|---|
| 1 — `#just-a-quick-ask` | ✅ Complete | ✅ Complete | ✅ Complete | ✅ Complete |
| 2 — `#engineering-pulse` | ✅ Complete | ✅ Complete | ✅ Complete | ✅ Complete |
| 3 — `#crit-aftermath` | ✅ Complete | ✅ Complete | ✅ Complete | ✅ Complete |

Decision 1 of Scenario 1 is fully written below. The code agent can wire that end-to-end immediately. Remaining content will be appended to this document as it's written.

---

## Data Structure

All scenario content lives in a single file: `data/scenarios.ts`.

```ts
export type ConflictStyle =
  | 'fighter'
  | 'negotiator'
  | 'diplomat'
  | 'avoider'
  | 'people-pleaser';

export type Message = {
  type: 'message';
  speaker: string;
  speakerInitials: string;
  avatarColor: string;
  text: string;
  timestamp: string;
  typingDelay?: number; // ms; default 2000
  alignment?: 'left' | 'right'; // 'right' for player messages
};

export type TimeMarker = {
  type: 'time-marker';
  label: string; // e.g. "— Friday, 8:47 AM —"
};

export type ChatItem = Message | TimeMarker;

export type DMIntercepted = {
  from: string;
  to: string;
  text: string;
};

export type ResponseOption = {
  style: ConflictStyle;
  label: string; // The response text shown on the button
  consequence: {
    chatItems: ChatItem[]; // Streams into the chat after the player's response
    dmIntercepted: DMIntercepted[]; // 1+ DMs shown in the slide-in panel
    futureYou: string; // Future You's coaching DM
  };
};

export type Decision = {
  id: string;
  incoming: ChatItem[]; // Messages that arrive before the player responds
  options: ResponseOption[]; // Always 5, one per conflict style
};

export type Scenario = {
  id: string;
  channelName: string; // e.g. "#just-a-quick-ask 🙃"
  channelTopic: string;
  difficulty: 'familiar' | 'tense' | 'personal';
  accentColor: string; // hex
  setupCard: {
    cast: { name: string; role: string }[];
    context: string;
    stakes: string;
  };
  decisions: Decision[]; // length 3
  summary: {
    costsByDominantStyle: Record<ConflictStyle, string>;
    earningsByDominantStyle: Record<ConflictStyle, string>;
  };
};
```

**Speaker style guide for avatars:**
- Devon → initials "DV", `avatarColor: '#FBBC04'`
- Maya → initials "MY", `avatarColor: '#34A853'`
- Sam → initials "SM", `avatarColor: '#5F6368'`
- Priya → initials "PR", `avatarColor: '#EA4335'`
- Riya → initials "RY", `avatarColor: '#4285F4'`
- You (player) → initials "AY", `avatarColor: '#202124'`, alignment `'right'`
- Future You → initials "AY+", `avatarColor: '#9333EA'`

---

## Scenario 1 — `#just-a-quick-ask 🙃`

### Setup Card

```
channelName: "#just-a-quick-ask 🙃"
channelTopic: "where PMs come to ask you small things at 4:47 PM on a Thursday."
difficulty: "familiar"
accentColor: "#FBBC04"  // Google Yellow

cast:
  - { name: "You", role: "Design Lead, 4 months in" }
  - { name: "Devon", role: "PM, peer, under pressure" }
  - { name: "Maya", role: "Senior IC designer, your report" }
  - { name: "Sam", role: "Your manager (may appear in some paths)" }

context: "The onboarding redesign ships Monday. Maya has been QA'ing for two weeks and flagged 12 issues including 4 P1s. It is Thursday, 4:47 PM."

stakes: "Ship Monday vs. ship right."
```

### Decision 1 — Devon's Opening Ask

> **Important — the SCENARIO SETUP card has been removed from the UI.** The context it used to carry (Thursday ~4:47 PM, onboarding ships Monday, leadership/demo pressure, Maya's two-week QA pass with ~12 issues / 4 P1s, ship-Monday-vs-ship-right) now surfaces **naturally inside this conversation** as Devon ramps up. Don't reintroduce an info-dump card. This lead-in is the only place that context lives.

**Incoming messages (stream in before player can respond):**

This is a real conversation unfolding, not an instant prompt. It builds in three beats — **light opener → Devon ramping into the pressure → the ask** — with **one non-scored player warm-up reply** in the middle to create two-way rhythm, kept short so the five options arrive quickly. (See the Code Agent note below for how the warm-up renders.)

```
1. TimeMarker: "— Thursday, 4:47 PM —"

2. Devon · 4:47 PM (typingDelay: 2000ms)
   "hey 👋 you around? not a fire. ok small lie. medium fire 🙃"
```

**[NON-SCORED warm-up beat]** — a single neutral player reply appears as ONE button (not the five conflict options):

```
3. You · 4:48 PM (alignment: right) (typingDelay: 0ms)
   "ha. hey — what's up?"
```

Clicking it streams Devon's tight ramp-up — two context beats, then the ask:

```
4. Devon · 4:48 PM (typingDelay: 2200ms)
   "onboarding redesign ships monday and leadership's been ALL over it — there's a demo, everyone wants it in the deck 😩"

5. Devon · 4:49 PM (typingDelay: 2200ms)
   "and maya's been deep in QA for ~2 weeks, just dropped ~12 issues on me. 4 P1s. but monday is monday, i can't move it 🫠"

6. Devon · 4:50 PM (typingDelay: 2200ms)
   "so honestly i think we just skip the QA pass and call it done. most of maya's stuff is nits right? we can patch after launch 🙏"
```

**Response Options:**

> **Notes for the Code Agent (Decision 1 lead-in):**
> - **Setup card removed.** Do not render the old cast/context/stakes card before this thread. The scenario's `setupCard` data can stay in `scenarios.ts` for reference/metadata, but it is **not displayed** at the top of the Decision 1 screen. All orientation now comes from the lead-in messages.
> - **Optional orientation time-marker:** item #1 (`— Thursday, 4:47 PM —`) is a normal `TimeMarker` chat item, kept as a light orientation cue in place of the deleted card. Drop it if it feels redundant — it's a nicety, not load-bearing.
> - **Non-scored warm-up beat (item #3):** render the opener (items #1–#2) first, then show **one** neutral button labeled `"ha. hey — what's up?"`. This is **NOT** one of the five conflict-style options and carries **no `style` and no scoring** — do not push anything to `choiceHistory`. On click: append it as a right-aligned player message (like any player turn), then stream Devon's ramp-up (#4–#6) using the normal typing/streaming rules. Only **after** #6 lands do the five scored `ResponseOption` buttons fade in. (If you prefer zero clicks, auto-advancing after a short beat is an acceptable fallback, but the single neutral button is the intended design — it makes the thread feel two-way before the real decision.)
> - **Suggested minimal type support:** an optional `warmup?: { playerLabel: string; afterMessages: ChatItem[] }` on `Decision`, where `playerLabel` is the neutral button text and `afterMessages` (#4–#6) stream in after it's clicked. Decisions without a warm-up omit the field. This keeps the warm-up out of the scored `options` array entirely.
> - **The five scored options below are unchanged** — Devon's culminating ask (#6) keeps the core ask (skip the QA pass / "most are nits right?" / patch after launch 🙏), so every existing consequence / DM / Future You still follows naturally from it.

---

#### 🔥 Fighter

**Button text:**
> "Devon — they're not nits. Maya documented 4 P1s and the rest aren't sundae sprinkles either. We've shipped two rough onboardings this year. I'm not making it three to make Monday feel good."

**Consequence chat items:**

```
1. (8-minute pause simulated: no typing indicator for 3000ms)
2. Devon · 4:55 PM (typingDelay: 0ms)
   "ok understood. i'll let leadership know we're slipping to wednesday. just fyi this isn't going to land well for either of us."
```

**DM intercepted:**

```
From: Devon
To: Priya
Text: "priya can u talk lol. design lead just shut down my QA-skip ask. so dramatic. third time this quarter. starting to think we need to rethink how design fits into launches here ¯\_(ツ)_/¯"
```

**Future You:**

> Hey. You held the craft line, and that mattered.
>
> But you also handed Devon the story that "design always pushes back." He's already writing the version of you he'll tell leadership about. The framework move you skipped was **Radical Candor** — challenge directly *while* caring personally. You did the challenge. You skipped the care. The phrase "for both of us" was on the table. You used it as a threat, not as an offer.
>
> Future you wants to ask: do you want to be right, or do you want to be respected? Sometimes those aren't the same answer.

---

#### ⚖️ Negotiator

**Button text:**
> "Okay, compromise: we cut QA on secondary flows (forgot password, social login) but keep the QA on the main email signup. That's where the P1s are. Deal?"

**Consequence chat items:**

```
1. Devon · 4:49 PM (typingDelay: 2000ms)
   "yeah ok that works. let me ping leadership and confirm. you're saving my ass on this one 🙏"

2. TimeMarker: "— 9:47 PM —"

3. Maya · 9:47 PM (typingDelay: 2000ms)
   "hey — devon just slacked the team we're skipping QA on secondary flows. did you sign off on that? I had a P0 in forgot-password I was going to flag tomorrow. were you going to ask me first?"
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "ngl i think my design lead just used my work as a bargaining chip without asking me. that doesn't feel great. is that normal? like am i being precious here"
```

**Future You:**

> You compromised. Sometimes that's the move.
>
> But you compromised someone else's work without consulting them — which makes it less of a compromise and more of a unilateral cut. **SBI** on this one: the *situation* was a deadline crunch, your *behavior* was scope reduction without consultation, the *impact* was Maya feeling thrown under the bus and a P0 still headed to prod.
>
> Next time you negotiate scope on someone else's work, pause Devon, talk to Maya, come back with a counter. Negotiation works. Negotiation without consultation is just managing up wearing a costume.

---

#### 🤝 Diplomat

**Button text:**
> "Before we skip QA — what does leadership actually need to see Monday? If it's the board demo, we might be able to ship a polished happy path and leave the rest for Tuesday. Hop on for 10?"

**Consequence chat items:**

```
1. Devon · 4:48 PM (typingDelay: 2000ms)
   "ok lemme check. give me 10"

2. TimeMarker: "— 5:19 PM —"

3. Devon · 5:19 PM (typingDelay: 2500ms)
   "okay so leadership actually just needs the email signup polished for the demo monday. social and forgot-password can come tuesday. that work for you?"
```

**DM intercepted (two DMs shown):**

```
DM 1:
From: Devon
To: Priya
Text: "okay turns out we were both panicking about the wrong thing. design lead is a beast — got me to actually ask leadership what they needed instead of just relaying their stress. she's the best."

DM 2:
From: Maya
To: Riya
Text: "btw design lead got us an extra day on social and forgot-password. i was about to lose my mind. she just made my week."
```

**Future You:**

> This was the highest-leverage move you could've made. You didn't accept the false binary. You asked what "ship" actually meant. **Thomas-Kilmann Collaborate quadrant** — both parties get what they need. The Radical Candor frame fits too: you cared about Devon's pressure *and* challenged directly on the framing.
>
> Most designers would have flinched and either fought or folded. You held.
>
> One risk you should know about: this move only works when the other person is operating in good faith. When they aren't — and you'll meet someone who isn't, eventually — this same approach gets played against you. File that away.

---

#### 🚪 Avoider

**Button text:**
> "Let me think about it and get back to you tomorrow. I want to check what Maya's actually found before I make that call."

**Consequence chat items:**

```
1. Devon · 4:48 PM (typingDelay: 2000ms)
   "ok but i need an answer by tomorrow morning. just fyi"

2. TimeMarker: "— Friday, 8:47 AM —"

3. Sam · 8:47 AM (typingDelay: 2000ms)
   "hey — devon mentioned you weren't sure about monday ship. can we talk?"
```

**DM intercepted:**

```
From: Devon
To: Priya
Text: "design lead avoiding the question again. why is it always pulling teeth with her. had to escalate to sam just to get a yes/no."
```

**Future You:**

> Deferring is a real strategy. Buying time when stakes are unclear can be wise.
>
> But you didn't say "I'll have an answer by 9 AM tomorrow, after I check three things." You said "let me think." Devon couldn't read whether you were thinking or stalling, so he assumed stalling and escalated. The framework move you missed: **specificity**.
>
> A specific defer ("by 9 AM tomorrow, after I talk to Maya") is delegation. An unspecific defer ("let me think") is avoidance. Sam DMing you at 8:47 AM is the cost of the ambiguity.

---

#### 🙏 People Pleaser

**Button text:**
> "Yeah, if leadership needs Monday we'll make it work. Most of Maya's stuff can probably wait. I'll let her know."

**Consequence chat items:**

```
1. Devon · 4:48 PM (typingDelay: 2000ms)
   "oh thank god. you're saving me here 🙏 letting the team know in #launches now"

2. TimeMarker: "— 11:47 PM —"

3. Maya · 11:47 PM (typingDelay: 2000ms)
   "i just saw devon's post in #launches that we're skipping QA. did we really? i have 4 P1s open. were you going to tell me?"
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "okay i think my design lead just sold me out to make the PM happy. like i found real bugs and they're just shipping to prod monday because devon was stressed. i'm so tired."
```

**Future You:**

> You took the path of least resistance with Devon. The cost landed on Maya, who wasn't even on the thread.
>
> This is the most common failure mode for new design leads — managing the upward relationship at the expense of the downward one. **AID frame**: your *action* was to agree without consulting, the *impact* was on Maya, the *do-differently* is to never accept scope changes on someone else's work without your team in the conversation.
>
> One principle to add to your kit, in your own handwriting: *I never agree to cut QA on my team's behalf without my team in the room.* Pin it somewhere.

---

### Decision 2 — Maya Turns to You

D1 was managing **up** to Devon. D2 is the **downward** relationship: Maya — your senior IC, your report, the person who actually found the bugs — is now standing in front of you. Whatever you decided with Devon, Maya is the one who has to live with it.

**The decision is shared.** Only the **first incoming line swaps** based on `choiceHistory[0]` — Maya arrives in a different mood depending on how D1 went. Everything after that first line, and all five response options, are the same across paths. (See *Notes for the Code Agent* for the `incomingVariants` shape.)

**Incoming messages (stream in before player can respond):**

The first message is a **framing variant** — pick by `choiceHistory[0]`:

```
VARIANT [if D1 = fighter] — grateful-but-wary
1a. Maya · 5:02 PM (typingDelay: 2500ms)
    "hey — heard you held the line with devon on the QA pass. thank you, genuinely. 🙏 i was bracing to get steamrolled again. ...is it weird that i'm also a little nervous about how he's going to be with us next sprint?"

VARIANT [if D1 = negotiator] — betrayed / blindsided (continues from her 9:47 PM question)
1b. Maya · 9:51 PM (typingDelay: 3000ms)
    "ok i've been staring at this for ten minutes. i'm not trying to be difficult. but you traded away QA on forgot-password — which is *my* area, with a P0 in it — and i found out from devon's channel post. so. were you going to ask me first, or am i finding the rest out from #launches too?"

VARIANT [if D1 = diplomat] — grateful / relieved
1c. Maya · 5:24 PM (typingDelay: 2500ms)
    "wait — devon just said we got an extra day on social and forgot-password?? i could cry a little. 😮‍💨 thank you for actually asking leadership what they needed instead of just passing the panic down. that's the first time that's happened to me here."

VARIANT [if D1 = avoider] — confused (Sam got pulled in)
1d. Maya · 9:03 AM (typingDelay: 2500ms)
    "hey, kind of confused — sam pinged me this morning asking what i'd 'actually found' for the monday ship? i didn't realize this was a sam-level thing. did something happen with devon yesterday? i feel like i missed a meeting i was supposed to be in."

VARIANT [if D1 = people-pleaser] — hurt / sold-out
1e. Maya · 11:58 PM (typingDelay: 3000ms)
    "i saw devon's #launches post. we're skipping QA monday. i have 4 P1s open that i documented for two weeks. i'm not going to pretend that doesn't sting — it feels like my work got handed away to make a deadline feel good, and nobody told me. i just need to understand: did you sign off on this?"
```

Then the **shared** continuation (same for every path):

```
2. Maya · [same minute as variant] (typingDelay: 2000ms)
   "sorry — i know you're getting it from all sides too. i just don't want to be the person who finds out what happened to my own work in a channel. can we talk about it like, actually?"
```

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Maya — I made the call. I'm the design lead; that's literally the job. I get that the rollout was messy, but I'm not going to relitigate every launch decision in a thread. Channel the energy at the deadline, not at me."

**Consequence chat items:**

```
1. (long pause: no typing indicator for 4000ms)
2. Maya · [+5 min] (typingDelay: 0ms)
   "got it. understood."
3. Maya · [+5 min] (typingDelay: 1500ms)
   "i'll update the bug tracker and tag it 'leadership-aware' so it's documented."
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "asked my lead to actually talk about it. got 'i'm the lead, that's the job.' cool cool. lesson learned — stop raising things, just document them so it's on record and move on. i was kind of excited to work under a designer for once. 🙃"
```

**Future You:**

> You pulled rank. Sometimes a leader does have to hold a decision without a town hall, and "I made the call" is a complete sentence. Authority that never gets used isn't authority.
>
> But look at what Maya just did: she didn't argue. She went quiet, and she started *documenting* instead of *raising*. That's the **Compete** quadrant of Thomas-Kilmann — you won the exchange and lost the channel. On the **Radical Candor** grid, you challenged directly but dropped "care personally" entirely, which lands as the bad quadrant: Obnoxious Aggression. The cost isn't this conversation. The cost is the next bug she *doesn't* bring you because she's learned it's not worth it.
>
> Future you wants to ask: when your best IC stops surfacing problems, who do you think finds them next — you, or your users?

---

#### ⚖️ Negotiator

**Button text:**
> "Okay. You're right that it landed badly. Here's the deal: you take point on the post-launch patch sprint — your scope, your priority order — and I'll get you the protected time for it and make sure your name's on the fix. Fair trade for a rough Thursday?"

**Consequence chat items:**

```
1. Maya · [+3 min] (typingDelay: 2500ms)
   "i mean... yeah, owning the patch sprint is good. i'd want that."
2. Maya · [+3 min] (typingDelay: 2500ms)
   "i guess i just notice we jumped straight to 'what do you get out of it' before anyone said the thing out loud. but ok. i'll take the deal."
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "got offered a deal — i run the patch sprint, get protected time, name on the fix. which is genuinely good?? but it's funny how it skipped the part where someone just says 'yeah that wasn't ok.' like i got a contract instead of an apology. i'll take the contract. i think."
```

**Future You:**

> Smart, and not cynical — you converted a grievance into ownership and gave Maya real upside. That's the **Compromise** quadrant: everyone walks away with something. With a peer, this is often the move.
>
> But Maya isn't a peer. She reports to *you*, and you negotiated with her like a vendor. The thing about trading with your own report is that it teaches her that access, time, and credit are things she has to *bargain* for — when those are things you're supposed to allocate as her manager anyway. **SBI**: the *situation* was a trust dent, your *behavior* was offering a transaction, the *impact* is she got the deliverable but not the repair — and she clocked the difference. She said it out loud.
>
> A trade resolves the issue. It doesn't rebuild the trust. Next time, try saying the free part — "that wasn't ok" — *before* you reach for the deal.

---

#### 🤝 Diplomat

**Button text:**
> "You're right, and I owe you a straight answer. I made a call under pressure that touched your work, and I didn't loop you in first — that's on me, not on you. Here's what I'm thinking we do about the open issues. But I want to hear your read first. Walk me through what you'd protect."

**Consequence chat items:**

```
1. Maya · [+2 min] (typingDelay: 2500ms)
   "...ok. thank you. that actually helps more than you know."
2. Maya · [+2 min] (typingDelay: 3000ms)
   "the two i'd die on are the forgot-password P0 and the screen-reader focus trap on signup. everything else i can triage post-launch with a clear conscience. give me 20 min and i'll send you a ranked list."
3. Maya · [+2 min] (typingDelay: 1500ms)
   "for what it's worth — i'll remember that you said 'that's on me.' people here usually don't."
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "ok so my lead actually owned it. said 'that's on me, not on you' and then asked what *i'd* protect. i'm a little stunned. that's the first time a manager here has done the unglamorous version instead of the spin. i'd run through a wall for that, honestly."
```

**Future You:**

> This is the repair. You restored her dignity by naming the gap before she had to pry it out of you, and then you handed her the pen — "what would you protect?" — which turns a victim back into an owner. **Thomas-Kilmann Collaborate**, and the full **Radical Candor** square: you cared personally *and* challenged the situation honestly. You'll feel the difference in how she shows up Friday.
>
> Two things to carry, though. One: this only works because you *meant* it. The same words, said as technique, curdle into manipulation faster than anything else in this kit — Maya can smell rehearsed contrition from across the building. Two: owning it sets a precedent. If "I'll bring you in" becomes "every call I make gets re-opened," you've traded decisiveness for endless consultation. Repair the trust, then still be the one who decides.
>
> You spent twenty minutes and a little ego. Cheapest trust you'll ever buy.

---

#### 🚪 Avoider

**Button text:**
> "Hey, I hear you, and you deserve a real conversation — not a Slack thread at the end of a brutal day. Let's grab fifteen minutes tomorrow morning before standup and actually do this properly. I don't want to give you a half-answer right now."

**Consequence chat items:**

```
1. Maya · [+3 min] (typingDelay: 2500ms)
   "...ok. tomorrow before standup."
2. Maya · [+3 min] (typingDelay: 2500ms)
   "i'll hold you to that, though. i've had a lot of 'let's talk tomorrows' here that turned into never."
3. TimeMarker: "— Friday, 8:31 AM —"
4. Maya · 8:31 AM (typingDelay: 2000ms)
   "morning. still good for the 15 before standup? 🙂"
```

**DM intercepted:**

```
From: Maya
To: Riya
Text: "lead pushed our talk to tomorrow before standup. could be a real 'let it cool down' thing, could be a 'hope she forgets' thing. genuinely can't tell yet. i sent a reminder so it can't quietly evaporate. fingers crossed it's the good version."
```

**Future You:**

> Cooling-off is real. Hard conversations do go better with sleep and lower cortisol, and refusing to do the repair badly *right now* can be wisdom, not dodging. The **Avoid** quadrant of Thomas-Kilmann is legitimate when the timing is genuinely wrong.
>
> But notice she had to add "I'll hold you to that," and then she set the reminder herself. To someone already hurt, a deferral reads as a *coin flip* — half the room hears "I respect this enough to do it right," the other half hears "I'm hoping the urgency dies overnight." You bought yourself a better conversation, but you also bought yourself a deadline you absolutely cannot miss: that 8:31 AM ping is a tripwire. Show up sharp, or this becomes the third "let's talk tomorrow" she stops believing in.
>
> The defer is only honest if the follow-through is non-negotiable. You just made tomorrow morning load-bearing.

---

#### 🙏 People Pleaser

**Button text:**
> "Oh god, Maya, I'm so sorry — you're completely right, this was awful, I should have protected you. Don't worry, I'll fix it. I'll go back to Devon tonight and get the QA pass reinstated, all of it, I promise. You shouldn't have to ship anything you're not proud of."

**Consequence chat items:**

```
1. Maya · [+2 min] (typingDelay: 2500ms)
   "oh — i mean, that's... a lot. you'd reinstate the whole pass? devon already told the team and leadership."
2. Maya · [+2 min] (typingDelay: 2500ms)
   "i wasn't asking you to blow it all up, i just wanted to be talked to. now i'm kind of worried i'm about to cause a whole thing for you?"
3. TimeMarker: "— 12:14 AM —"
4. Devon · 12:14 AM (typingDelay: 2500ms)
   "wait i'm confused — maya says we're reinstating full QA now?? i already locked monday with leadership two hours ago. what's happening 😩"
```

**DM intercepted (two DMs shown):**

```
DM 1:
From: Maya
To: Riya
Text: "i asked for a conversation and got a whole apology spiral + a promise to undo everything? now i feel like i have to manage HER feelings about my feelings. i didn't want a grand gesture. i wanted thirty seconds of 'that wasn't ok.' it's weirdly more exhausting."

DM 2:
From: Devon
To: Priya
Text: "lol design lead is now trying to REVERSE the thing she agreed to this afternoon, at midnight, after i already told leadership it's locked. love that for me. how do i not look insane in standup tomorrow 🙃"
```

**Future You:**

> Your heart is in exactly the right place, and that matters more than you'll let yourself believe. But watch what over-correcting actually did: you made a promise that isn't yours to keep. The QA pass is already locked with leadership — you can't un-ring that bell by midnight DM, so now you've created a *second* mess on top of the first, and Maya is suddenly managing your guilt instead of receiving your support. This is the **Accommodate** quadrant: you sacrificed your own position (and your credibility with Devon) to make the discomfort go away.
>
> **AID**: your *action* was over-apologizing and over-promising a reversal you can't deliver; the *impact* is a flailing thread, a confused PM, and a report who now feels responsible for *your* spiral; the *do-differently* is to separate the apology from the fix. "I should have looped you in — that part's on me" costs you nothing and lands clean. "I'll reverse a locked decision tonight" costs you everything and lands as chaos.
>
> The apology was free and true. The promise was expensive and false. You only needed the first one — and you'll need that spent credibility on Friday.

### Decision 3 — Friday Standup, Leadership Walks In

It goes **public.** Friday morning, the team standup. Maya is here. Devon is here. And then — unannounced — a Director joins the call.

**Incoming messages (stream in before player can respond):**

```
1. TimeMarker: "— Friday, 9:30 AM · #launches —"
2. Sam · 9:30 AM (typingDelay: 2000ms)
   "morning all — quick heads up, Priya from leadership is sitting in on standup today to get a read on the Monday onboarding ship. nothing scary, she just wants status. 🙂"
3. Priya (Director) · 9:31 AM (typingDelay: 2500ms)
   "Hi team! 👋 Won't take much of your time. I'm presenting the onboarding launch to the exec staff at 11 today, so I just need a clear yes/no: are we fully ready to ship Monday? Anything I should know before I put it on a slide?"
4. Devon · 9:32 AM (typingDelay: 2000ms)
   "morning! good question for design i think 👀"
5. Maya · 9:32 AM (typingDelay: 1500ms)
   "👀"
```

> **Note for code agent:** This Priya is the **Director** (leadership), distinct from Devon's gossipy PM friend Priya referenced in DMs. Use the same avatar (`PR`, `#EA4335`) but the in-channel name should render as **"Priya (Director)"** to disambiguate. The room is watching: the player's answer goes to an exec slide, with Maya and Devon present. This is where D1 and D2 compound — see *compounding modifiers* and *lock conditions* per option below, and the consolidated table in *Notes for the Code Agent*.

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Honest answer, Priya: 'fully ready' is the wrong question. We found real defects this week. If the exec ask is a guaranteed-clean Monday, I can't put my name on that — and I'd rather tell you now than apologize on Tuesday."

**LOCK CONDITION:** *Never locked.* (You can always choose to push back — the only question is whether the room backs you, which depends on prior choices.)

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 9:34 AM (typingDelay: 3000ms)
   "Okay — I appreciate the candor, that's exactly why I sat in. Walk me through what 'not ready' means concretely."

CONDITIONAL — append based on history:

[if choiceHistory[1] = diplomat]  (you repaired with Maya)
2. Maya · 9:35 AM (typingDelay: 2000ms)
   "i can speak to that — there are two issues i'd protect: a forgot-password P0 and a screen-reader focus trap. everything else is genuinely post-launch-able. my lead and i ranked it yesterday."
3. Priya (Director) · 9:36 AM (typingDelay: 2500ms)
   "That's the most useful thing I've heard all week. A ranked list I can take to execs. Thank you both."

[if choiceHistory[1] = fighter OR choiceHistory[1] = people-pleaser]  (you burned or smothered Maya)
2. (pause: no typing indicator for 4000ms)
3. Maya · 9:36 AM (typingDelay: 0ms)
   "i'd have to pull the tracker to say which are blockers."  ← pointedly neutral; she does not back you in the room
4. Priya (Director) · 9:37 AM (typingDelay: 2500ms)
   "Hm. I'm hearing 'not ready' from you but 'unclear' from the team. I need you two aligned before 11."

[if choiceHistory[0] = fighter]  (you already fought Devon to Wednesday in D1)
5. Priya (Director) · 9:38 AM (typingDelay: 2500ms)
   "I'll be honest — this is the second time design has been the thing standing between me and a date this quarter. I trust the call. I also need to understand why it keeps being design. Let's talk after."
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Sam
To: (you)
Text: "that was a clean save in there. maya backing you with an actual ranked list made you look like a team, not a blocker. whatever you did with her yesterday — keep doing that."

[otherwise]
From: Priya (Director)
To: Sam
Text: "Sam — I like that your design lead has a spine, truly. But the team didn't echo the read, and that's a flag. Either the bugs are real and design isn't communicating them, or design is gold-plating. Help me figure out which before I'm on the wrong side of an exec slide."
```

**Future You:**

> You told leadership the truth to their face, in public, with your name on it. That is the rarest and most expensive thing a new lead can do, and there's a version of your career where this is the moment people started trusting your word. **Compete** quadrant, used on the *right* target this time — the date, not your report.
>
> But the public stand is only as strong as the room behind you. If you repaired with Maya yesterday, she just turned your "no" into a *credible, specific* no — that's the whole game. If you didn't, your conviction hangs in the air alone and reads as one person's opinion against a deadline. Conviction without corroboration is just volume.
>
> The lesson isn't "don't fight leadership." It's "earn the people who'll fight beside you *before* you need them." Friday's standing was set on Thursday.

---

#### ⚖️ Negotiator

**Button text:**
> "Here's a clean path, Priya: we ship the email-signup happy path Monday — polished, demo-ready for your exec deck — and we fast-follow the secondary flows Tuesday once the last checks clear. You get your Monday headline, we don't ship anything broken. Phased, on the record."

**LOCK CONDITION:** *Never locked,* but the framing shifts (see conditional). If D1 already carved the scope (`choiceHistory[0]` = negotiator or people-pleaser), you're not *proposing* a phase — you're *formalizing the cut you already made*, and the consequence reflects that.

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 9:34 AM (typingDelay: 3000ms)
   "A phased ship I can sell. 'Monday headline, Tuesday completion' actually slides nicely. Let me make sure the team's on the same page on what's in phase one."

CONDITIONAL:

[if choiceHistory[0] = diplomat]  (you already negotiated the extra day honestly in D1)
2. Maya · 9:35 AM (typingDelay: 2000ms)
   "this matches what we already scoped yesterday — email signup monday, social + forgot-password tuesday. it's real, not a band-aid. i'm good with this."
3. Priya (Director) · 9:36 AM (typingDelay: 2000ms)
   "Even better — so this is already the plan, not a new ask. Done. 👍"

[if choiceHistory[0] = negotiator OR choiceHistory[0] = people-pleaser]  (QA was cut, not scoped)
2. (pause: no typing indicator for 3500ms)
3. Maya · 9:36 AM (typingDelay: 0ms)
   "just so phase one is honest on the slide — the email-signup flow still has open P1s. 'polished happy path' is doing some lifting there."
4. Priya (Director) · 9:37 AM (typingDelay: 2500ms)
   "Okay — I want the phase-one boundary drawn around what's actually tested, not what we wish were tested. Redraw it and send me the real line by 10:30."

[if choiceHistory[1] = negotiator]  (you also traded with Maya in D2)
5. Maya · 9:38 AM (typingDelay: 2500ms)
   "noticing we're once again deciding the shape of my work in a room and i'm finding the edges of it live. i'll make phase one honest. but can the next scope call start with me in it?"
```

**DM intercepted:**

```
[if choiceHistory[0] = diplomat]
From: Devon
To: Priya (the PM friend)
Text: "design lead just turned our messy thursday into a clean 'phased ship' story in front of leadership and made us both look good. ngl i was bracing for blame and got cover instead. she's good in a room."

[otherwise]
From: Maya
To: Riya
Text: "watched a real cut get re-labeled a 'phased ship' to leadership in real time. i don't even think it's a lie exactly? it's just... the marketing version of the truth. i flagged the open P1s so it'd at least be on the record. small acts of resistance lol."
```

**Future You:**

> This is genuinely skilled. You reframed a binary ("ready / not ready") into a sequence ("Monday / Tuesday"), which is the **Compromise** quadrant at its best — leadership gets a headline, the team doesn't ship garbage, and nobody loses face. When the prior week was honest, this is just naming the plan, and Maya backs it instantly.
>
> The risk is the gap between *phasing* and *spin*. If D1 already cut QA, "polished happy path" isn't a phase — it's a cut wearing a nicer outfit, and Maya will quietly draw the real line in front of leadership so the record stays honest. That's her protecting *you* from a Tuesday you can't see yet. And if you also traded with her in D2, she'll name the pattern out loud: decisions about her work keep getting made in rooms she finds out about. The negotiation worked on leadership. The person it keeps costing you is the one redrawing your slide.
>
> A compromise is honest when the parts are real. Make sure "phase one" is a boundary, not a euphemism.

---

#### 🤝 Diplomat

**Button text:**
> "Let me give you something you can actually stand behind, Priya. Monday is real for the core flow — email signup, demo-ready. There are two issues we're deliberately holding for a Tuesday fast-follow, and we made that call on purpose, not by accident. I'll send you a one-pager so the exec story is 'shipped with eyes open,' not 'shipped and hoping.'"

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[0]` = negotiator OR people-pleaser** *(you cut the QA pass on Thursday)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't sell 'shipped with eyes open' — you cut the QA pass Thursday, so there's no clean inspection to point to. The honest-and-prepared story isn't available to you anymore. (It was, on Wednesday.)"*

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 9:34 AM (typingDelay: 3000ms)
   "'Shipped with eyes open' — yes. That's the framing I want in front of execs. A deliberate, documented decision reads as competence; a surprise reads as a miss. Send the one-pager."

CONDITIONAL:

[if choiceHistory[1] = diplomat]  (Maya repaired)
2. Maya · 9:35 AM (typingDelay: 2000ms)
   "i can have the one-pager to you in 30 — i already ranked the hold-for-tuesday list yesterday. forgot-password P0 and the screen-reader focus trap, with mitigations noted."
3. Priya (Director) · 9:36 AM (typingDelay: 2000ms)
   "This is what 'in control' looks like. Thank you, both of you."

[if choiceHistory[1] = avoider]  (you deferred Maya to this morning)
2. Maya · 9:35 AM (typingDelay: 2500ms)
   "we were literally about to sync on exactly this before standup — give us 30 after and you'll have the ranked one-pager."
3. Sam · 9:36 AM (typingDelay: 2000ms)
   "good — glad that convo's happening. 🙂"

[if choiceHistory[1] = fighter]  (you shut Maya down in D2)
2. (pause: no typing indicator for 3500ms)
3. Maya · 9:36 AM (typingDelay: 0ms)
   "the issues are in the tracker, tagged 'leadership-aware.' i'll let my lead speak to which we're holding."  ← accurate, correct, and notably not warm
4. Priya (Director) · 9:37 AM (typingDelay: 2500ms)
   "Okay. The plan's good. The temperature between you two I'll leave to Sam."
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Maya
To: Riya
Text: "we just walked into leadership *aligned*, with a ranked list, and called the cut a deliberate decision instead of a screwup — because it WAS one. i've never felt this un-thrown-under-a-bus at work. i'd follow this person somewhere."

[if choiceHistory[1] = fighter]
From: Maya
To: Riya
Text: "covered for the plan in standup because it's the right plan and i'm a professional. did not go one degree warmer than that. 'leadership-aware' tag is carrying a lot of quiet weight today lol."
```

**Future You:**

> "Shipped with eyes open." You gave leadership the one thing execs actually want — not a clean story, a *controlled* one — and you did it without throwing your team or the truth under the bus. **Collaborate** quadrant, full **Radical Candor** square, in the highest-stakes room of the week. This is the ceiling of the whole scenario.
>
> But notice it was only *available* to you because of Wednesday. If you'd cut QA on Thursday, this door was locked — you can't sell "deliberate and documented" when the documentation is the thing you skipped. And notice who makes the one-pager land: if you repaired with Maya, she co-signs it in 30 minutes and you look like a unit; if you shut her down, she'll still cover the *plan* — because she's a professional — but she won't spend a single degree of warmth she doesn't owe you, and leadership will feel the chill even if they can't name it.
>
> The best move in the room was set up days earlier. That's the whole game: leadership sees the standup, but the standup was decided on Thursday afternoon.

---

#### 🚪 Avoider

**Button text:**
> "Good question — there are a few moving pieces I want to confirm before I give you a number I'd stake the slide on. Let me pull the latest from the tracker and follow up with you offline well before 11, rather than guess in standup."

**LOCK CONDITION:** *Never locked,* but the room's patience depends on history (see conditional). After a week of deferring, the same move reads very differently.

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 9:34 AM (typingDelay: 3000ms)
   "Sure — offline's fine, as long as 'offline' means before 10:30, because my deck doesn't write itself. 🙂"

CONDITIONAL:

[if choiceHistory[0] = avoider]  (you also deferred Devon in D1)
2. Sam · 9:35 AM (typingDelay: 2500ms)
   "i'll grab the follow-up with you so it actually happens — no offense, but this is the second 'let me follow up' on this ship and Priya needs a real answer this time."
3. Priya (Director) · 9:36 AM (typingDelay: 2000ms)
   "Appreciated, Sam. I need a yes/no with reasons by 10:30, not another placeholder."

[if choiceHistory[1] = avoider]  (you also deferred Maya in D2)
4. Maya · 9:36 AM (typingDelay: 2500ms)
   "happy to send the tracker read in 15 if it helps — i've got the P1 list ready to go."  ← she's quietly doing the thing you keep deferring

[if choiceHistory[1] = diplomat]  (you repaired Maya)
2. Maya · 9:35 AM (typingDelay: 2000ms)
   "i can shortcut this — we ranked it yesterday. i'll drop the list in the offline thread so the follow-up's basically pre-written."
```

**DM intercepted:**

```
[if choiceHistory[0] = avoider OR choiceHistory[1] = avoider]
From: Priya (Director)
To: Sam
Text: "Sam, real talk — your lead keeps reaching for 'let me follow up.' Once is prudence. Three times on one launch is a pattern, and I can't put 'TBD' on an exec slide. I need to know if this is someone who's careful or someone who can't commit. Which is it?"

[otherwise]
From: Maya
To: Riya
Text: "lead punted leadership to 'offline before 11.' which... is actually reasonable? you shouldn't guess a ship date in front of execs. i just hope offline-before-11 is a real plan and not a vibe. i dropped my P1 list in the thread so there's something concrete to follow up WITH."
```

**Future You:**

> Refusing to invent a number in front of an exec is a genuinely defensible instinct — a guessed "yes" you can't back is worse than an honest "let me confirm." The **Avoid** quadrant is correct when the cost of a wrong commitment is high and the information is one tracker-pull away.
>
> But avoidance has a *counter*, and you may have already triggered it: a single defer is prudence; a third defer on the same launch is a reputation. If you punted Devon in D1 and Maya in D2, leadership now has a pattern, and Sam stepping in to "make sure it happens" is the sound of your manager quietly removing your discretion. Worse — notice who keeps filling the vacuum. Maya, every time, is offering the concrete answer you keep declining to give. The work you're avoiding doesn't disappear; it just gets done by someone with less authority and more frustration.
>
> A defer is a loan against your credibility. You can carry one. By the third, leadership starts asking Sam whether you're careful or just can't commit — and that's a question you never want asked *about* you instead of *to* you.

---

#### 🙏 People Pleaser

**Button text:**
> "Yes! We're good for Monday, Priya — the team's been heads-down all week and it's in great shape. You can absolutely put 'ready to ship' on the slide. We've got this. 🙌"

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[0]` = negotiator OR people-pleaser** *(you cut the QA pass on Thursday)* **AND Maya is in the room** *(she always is in D3)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't claim it's tested — you cut the QA pass on Thursday, and Maya is in this room with the open P1 list. Saying 'fully ready' out loud now is a lie she can see. (Pick a story you can survive at 11 AM.)"*

> **Design note:** This is the central Bandersnatch lock. The "tell them what they want to hear" reflex is *removed from the board* the moment your own earlier shortcut would make it a provable, public lie with a witness. If D1 was Fighter/Diplomat/Avoider (QA not cut), the option is available — but it's still a costly choice, not a free one (see below).

**Consequence chat items (only reachable when NOT locked — i.e. `choiceHistory[0]` ∈ {fighter, diplomat, avoider}):**

```
BASE:
1. Priya (Director) · 9:34 AM (typingDelay: 2500ms)
   "Love that energy — 'ready to ship' it is. I'll put it on the slide. 🎉"

CONDITIONAL:

[if choiceHistory[0] = diplomat]  (extra day exists; you're overstating, not lying)
2. Maya · 9:35 AM (typingDelay: 2500ms)
   "small flag before it hits a slide — we'd scoped social + forgot-password for tuesday, not monday. monday's the email flow. don't want 'fully ready' to over-promise the parts we deliberately moved."
3. Priya (Director) · 9:36 AM (typingDelay: 2000ms)
   "Ah — good catch. 'Core ready Monday, rest Tuesday.' Glad we said it now and not at 11. 🙂"

[if choiceHistory[0] = fighter]  (you fought to Wednesday — now you're publicly contradicting your own week-ago self)
2. Devon · 9:35 AM (typingDelay: 2500ms)
   "wait — monday?? i told leadership wednesday on tuesday because design said it wasn't ready. now it's ready? 😵‍💫 which is it"
3. Priya (Director) · 9:36 AM (typingDelay: 2500ms)
   "I'm getting whiplash. Tuesday it was Wednesday, today it's Monday. I need *one* answer from design that survives until 11."

[if choiceHistory[1] = fighter OR choiceHistory[1] = negotiator]  (Maya isn't invested enough to save you)
4. Maya · 9:37 AM (typingDelay: 3000ms)
   "i'll just note the tracker still shows open P1s. not my call what goes on the slide."  ← she will not co-sign the optimism
```

**DM intercepted:**

```
[if choiceHistory[0] = diplomat]
From: Maya
To: Riya
Text: "lead got a little exuberant and almost let 'fully ready' onto an exec slide, but caught the over-promise when i flagged it. enthusiasm slightly outran the facts but we landed honest. i'll take it."

[otherwise]
From: Devon
To: Priya (the PM friend)
Text: "the date has now been 3 different days in 3 days depending on who's stressed and who's in the room. i no longer know what we're shipping monday and i'm the PM. send help 🙃"
```

**Future You:**

> When this option is even *available* to you, the impulse is the kindest one in the room — you wanted to give Priya relief and the team a win. Hold onto that instinct; it's not the problem. The delivery is.
>
> "Fully ready" is a check you're writing against Monday's account, and you don't control Monday's balance. **Accommodate** quadrant: you bought the room's comfort with your own credibility. If you got the extra day in D1, Maya catches the over-promise and softens it to "core Monday, rest Tuesday" — she saves you, *this time*, because you didn't burn her. If you fought to Wednesday a week ago, you're now publicly contradicting your own past self and Devon says so out loud, in front of the Director. And if you cut QA Thursday, you couldn't even reach this option — the lie was too visible to say.
>
> **AID**: the *action* was telling leadership the comfortable thing; the *impact* is an exec slide that may not survive contact with Monday; the *do-differently* is to give relief with a *true* sentence — "core flow's ready, two items land Tuesday" delivers 90% of the warmth and 0% of the Tuesday hangover.
>
> The kindest thing you can say to leadership is the thing that's still true at 11 AM. Comfort that expires is just deferred panic with better manners.

---

### Summary Content — Performance Review

**Dominant style** = the mode of `choiceHistory` across the three decisions. Ties broken by frequency in the order **Diplomat > Negotiator > Fighter > Avoider > People Pleaser** (per *Notes for the Code Agent*). The two lines below feed §2.3's "What it cost you" / "What it earned you," written in Future You's voice.

```ts
summary: {
  costsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
  earningsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
}
```

#### 🔥 Fighter — dominant

**What it cost you:**
> You spent the week being right *at* people instead of *with* them. Every line you held, you held alone — and the bill for that came due in the standup, when conviction turned out to be a solo instrument. Somewhere in there Maya stopped raising problems and started just documenting them, which is the quiet sound of a great IC deciding you're not worth the friction. That's the **Compete** quadrant's tax: you win the exchanges and slowly lose the room.

**What it earned you:**
> A spine that people can feel, which is rarer than you think and worth real money. Devon knows you won't fold under a deadline, leadership knows your "no" means something, and craft got protected when it would've been cheap to let it slide. You were the one who said the true thing in the room with your name on it. The work now is to keep the directness and add back the half of **Radical Candor** you keep skipping — care personally — so the next person fights *beside* you instead of just respecting you from a distance.

#### ⚖️ Negotiator — dominant

**What it cost you:**
> You turned a lot of conflicts into deals, and a couple of those deals were with people who shouldn't have had to bargain with you — Maya, mostly, who kept noticing that the shape of her work got decided in rooms and traded across tables she wasn't sitting at. You got the deliverable and skipped the repair more than once. **SBI** would call it: the trades were sound, but a few of them spent trust to buy time, and trust is the more expensive currency.

**What it earned you:**
> You're the person who finds the third option when everyone else is stuck on two. "Monday headline, Tuesday completion" is the kind of reframe that makes leadership exhale and teams keep their dignity — that's the **Compromise** quadrant working as designed. You move conflicts toward outcomes instead of standoffs. Carry that. Just remember some moments don't want a deal; they want you to say the free, true sentence first — "that wasn't ok" — *before* you reach for the contract.

#### 🤝 Diplomat — dominant

**What it cost you:**
> Time, and a little of the decisiveness people also need from a lead. Owning the gap and bringing Maya in was the right call — but if "I'll loop you in" hardens into "every decision gets re-opened," you've traded being trusted for being endlessly consulted. **Collaborate** is the most expensive quadrant to run: it works beautifully and it does not scale to every conflict. Some calls you just have to make and wear.

**What it earned you:**
> The strongest thing in the building: a team that walks into the highest-stakes room *aligned*, because you did the unglamorous repair days before anyone was watching. You named your own gaps before they were pried out of you, handed people back the pen, and made "shipped with eyes open" a story leadership could actually stand behind. Maya would run through a wall for you, and that's not sentiment — that's the corroboration that turned your standup "no" into a credible one. You proved the best move in the room was set up on Thursday afternoon.

#### 🚪 Avoider — dominant

**What it cost you:**
> A reputation you didn't mean to build. One "let me follow up" is prudence; you reached for it enough times on one launch that leadership started asking Sam whether you're *careful* or just *can't commit* — and that's a question you never want asked about you instead of to you. The **Avoid** quadrant is a loan against your credibility, and you carried a balance. Worse: every vacuum you left, Maya quietly filled, doing the deciding you kept deferring, with less authority and more frustration than either of you needed.

**What it earned you:**
> You almost never made things worse by reacting too fast, and that's not nothing — you refused to invent a ship date in front of execs, and a guessed "yes" you can't back really is worse than an honest "let me confirm." When the information was one tracker-pull away and the cost of a wrong commit was high, waiting was wisdom. The fix is small and entirely within reach: make every defer *specific* — "by 10:30, after I check three things." A specific defer is delegation. An unspecific one is the thing leadership started worrying about.

#### 🙏 People Pleaser — dominant

**What it cost you:**
> You kept buying the room's comfort with your own credibility, and that account ran low by Friday. The midnight promise to Maya you couldn't keep, the "fully ready!" that didn't survive contact with the facts — each one felt like kindness in the moment and landed as a mess someone else had to clean up, sometimes Maya, sometimes Devon, sometimes you at 11 AM. **Accommodate** is the quadrant where you sacrifice your own position to make discomfort disappear, and discomfort, it turns out, does not disappear. It compounds.

**What it earned you:**
> Genuine warmth, and people felt it — your instinct to protect others and absorb the friction is the raw material of every leader worth following. Don't kill it; aim it. The lesson the week kept teaching is that the apology and the fix are two different objects: "I should have looped you in" costs nothing and lands clean, while "I'll reverse the locked decision tonight" costs everything and lands as chaos. **AID** in one line — keep the care, separate it from the over-promise, and give people relief with a sentence that's still true tomorrow. The kindest thing you can say is the thing that survives until morning.

---

## Notes for the Code Agent — Scenario 1 (Decisions 2 & 3)

Everything in this section is what the build needs that goes *beyond* the Decision 1 pattern. Decisions 2 and 3 introduce **branching**, which requires two small, optional, backwards-compatible additions to the TS types. Decision 1 uses none of these fields and is unaffected.

### (a) Decision 2 — `incoming` framing-variant swap keyed to `choiceHistory[0]`

D2 has **one shared decision**. Only the **first incoming message** changes based on the D1 choice; message #2 and all five options are identical across paths.

**Proposed minimal type addition** (optional field on `Decision`):

```ts
export type Decision = {
  id: string;
  incoming: ChatItem[];                 // shared; used as-is when no variant applies
  incomingVariants?: Partial<           // OPTIONAL — D2 only
    Record<ConflictStyle, ChatItem[]>
  >;                                     // keyed by choiceHistory[0]
  options: ResponseOption[];
};
```

**Build rule for D2:** if `incomingVariants[choiceHistory[0]]` exists, render it **first**, then render the shared `incoming` array (which holds message #2, the "can we talk about it like, actually?" line). If no variant matches, render `incoming` alone. The five variant blocks are labelled in the doc as `VARIANT [if D1 = …]`.

| `choiceHistory[0]` | Maya's mood on entry | variant id |
|---|---|---|
| `fighter` | grateful-but-wary | `1a` |
| `negotiator` | betrayed / blindsided (continues her 9:47 PM question) | `1b` |
| `diplomat` | grateful / relieved | `1c` |
| `avoider` | confused (Sam pulled her in) | `1d` |
| `people-pleaser` | hurt / sold-out | `1e` |

> Timestamps in the variants intentionally differ (5:02 PM after the Fighter afternoon, 9:51 PM after the late-night Negotiator, Friday 9:03 AM after the Avoider morning, etc.) to keep the timeline continuous with where each D1 path ended. The shared message #2 uses "[same minute as variant]" — stamp it equal to the variant's timestamp.

### (b) Decision 3 — lock conditions + compounding modifiers keyed to `choiceHistory`

D3's `incoming` is **fully shared** (no variants). The branching lives **per option**, in two forms:

1. **Lock conditions** — an option can be disabled with greyed-out copy.
2. **Compounding modifiers** — when an option *is* chosen, its consequence `chatItems` / `dmIntercepted` / (occasionally) `futureYou` framing changes based on `choiceHistory`. The doc writes these as labelled conditional sub-blocks: `BASE:` (always shown) followed by `[if choiceHistory[…] = …]` appendices.

**Proposed minimal type additions** (both OPTIONAL, both on `ResponseOption`):

```ts
// A predicate over the choices made so far. Pure, serializable-as-data is fine too,
// but a function keeps the doc's conditions readable 1:1.
export type ChoicePredicate = (choiceHistory: ConflictStyle[]) => boolean;

export type ConditionalBlock = {
  when: ChoicePredicate;                 // appended only if true
  chatItems?: ChatItem[];
  dmIntercepted?: DMIntercepted[];       // replaces base DM when matched (see rule)
};

export type ResponseOption = {
  style: ConflictStyle;
  label: string;
  lockedWhen?: ChoicePredicate;          // OPTIONAL — if true, render disabled
  lockedReason?: string;                 // OPTIONAL — greyed-out copy shown in place of the button
  consequence: {
    chatItems: ChatItem[];               // the BASE block
    conditionalChatItems?: ConditionalBlock[];   // OPTIONAL — appended in array order where when() is true
    dmIntercepted: DMIntercepted[];      // the default/BASE DM(s)
    conditionalDM?: ConditionalBlock[];  // OPTIONAL — first matching block's dmIntercepted REPLACES base
    futureYou: string;
  };
};
```

**Consequence assembly rule for D3:**
1. Render `consequence.chatItems` (the `BASE:` block).
2. For each entry in `conditionalChatItems` in array order, if `when(choiceHistory)` is true, append its `chatItems`. (Multiple can stack — e.g. a Fighter pick can append both a "Maya backs you" block *and* a "second time it's design" leadership block.)
3. For DMs: walk `conditionalDM` in order; the **first** block whose `when()` is true supplies the DM panel. If none match, use base `dmIntercepted`. (The doc's `[otherwise]` DM = base `dmIntercepted`.)
4. `futureYou` is static per option (the coaching already references all branches in prose).

**Lock conditions defined (3 meaningful locks across the five options):**

| Option | `lockedWhen` (choiceHistory) | `lockedReason` (greyed-out copy) |
|---|---|---|
| 🔥 Fighter | *(never)* | — |
| ⚖️ Negotiator | *(never)* | — (framing shifts via `conditionalChatItems`, but always selectable) |
| 🤝 Diplomat | `choiceHistory[0] === 'negotiator' \|\| choiceHistory[0] === 'people-pleaser'` | "🔒 You can't sell 'shipped with eyes open' — you cut the QA pass Thursday, so there's no clean inspection to point to. The honest-and-prepared story isn't available to you anymore. (It was, on Wednesday.)" |
| 🚪 Avoider | *(never)* | — (room patience shifts via `conditionalChatItems`) |
| 🙏 People Pleaser | `choiceHistory[0] === 'negotiator' \|\| choiceHistory[0] === 'people-pleaser'` | "🔒 You can't claim it's tested — you cut the QA pass on Thursday, and Maya is in this room with the open P1 list. Saying 'fully ready' out loud now is a lie she can see. (Pick a story you can survive at 11 AM.)" |

> Net effect: a D1 QA-cut (`negotiator` or `people-pleaser`) **closes two doors** in D3 (Diplomat *and* People Pleaser), leaving Fighter / Negotiator / Avoider — i.e. once you've cut, you can only fight, formalize the cut, or stall; you can no longer claim the clean story or the comfortable lie. This is the intended Bandersnatch lock. **Edge case:** ensure at least the three never-locked options remain selectable in every path so the decision is never a dead end (it always is — Fighter, Negotiator, Avoider are never locked).

**Compounding modifier index (which prior choice each option's conditionals read):**

| Option | reads `choiceHistory[0]` (D1) for… | reads `choiceHistory[1]` (D2) for… |
|---|---|---|
| 🔥 Fighter | "second time it's design" leadership line (`[0]=fighter`) | whether Maya backs you (`[1]=diplomat`) vs. stays neutral (`[1]=fighter`/`people-pleaser`) |
| ⚖️ Negotiator | phase vs. spin framing + Maya's honesty flag (`[0]=diplomat` vs. `negotiator`/`people-pleaser`) | "deciding my work in a room again" line (`[1]=negotiator`) |
| 🤝 Diplomat | *(gate only — see lock)* | one-pager co-sign warmth: `[1]=diplomat` (warm) / `avoider` (sync pending) / `fighter` (cold-but-correct) |
| 🚪 Avoider | Sam steps in on repeat-defer (`[0]=avoider`) | Maya fills the vacuum (`[1]=avoider`) / shortcuts it (`[1]=diplomat`) |
| 🙏 People Pleaser | *(gate only for the lie; when available)* over-promise softened (`[0]=diplomat`) / public contradiction (`[0]=fighter`) | Maya won't co-sign optimism (`[1]=fighter`/`negotiator`) |

### (c) Disambiguation: two Priyas

- **Priya (Director)** = leadership, appears **in-channel** in D3. Render display name as **"Priya (Director)"**. Reuses the existing Priya avatar (`PR`, `#EA4335`).
- **Priya (the PM friend)** = Devon's gossipy DM contact from D1/D2 DMs. Only ever appears as a **DM recipient**, never in-channel. In D3 DM blocks she's labelled "Priya (the PM friend)" to keep the two separate. Same avatar is fine since they never co-occur in the same surface.

### (d) Timeline continuity (for sanity-checking timestamps)

- D1 path end-states: Fighter ~4:55 PM Thu · Negotiator ~9:47 PM Thu · Diplomat ~5:19 PM Thu · Avoider Fri 8:47 AM · People-Pleaser ~11:47 PM Thu.
- D2 (Maya conversation) picks up from each of those (5:02 PM / 9:51 PM / 5:24 PM / Fri 9:03 AM / 11:58 PM respectively).
- D3 (Friday standup) is **Fri 9:30 AM** for all paths — by Friday morning every thread has converged on the same standup, regardless of when D1/D2 happened. This convergence is intentional and lets D3 be a single shared `incoming`.

---

## Scenario 2 — `#engineering-pulse 📈`

### Setup Card

> **Note:** This `setupCard` is **metadata only** — per the current UI, the visible setup card is removed. The context surfaces naturally in Decision 1's conversational lead-in. Keep the data for reference.

```
channelName: "#engineering-pulse 📈"
channelTopic: "where the dashboards are green and the conversations are not."
difficulty: "tense"
accentColor: "#4285F4"  // Google Blue

cast:
  - { name: "You", role: "Design Lead" }
  - { name: "Theo", role: "Eng lead, peer, chasing a perf target" }
  - { name: "Nadia", role: "Accessibility lead, peer — will defer to your call" }
  - { name: "Marcus", role: "Marketing, already drafting the launch post" }

context: "The new dashboard ships in 48 hours. Eng is 180ms over the perf budget. Theo wants to cut the accessibility layer — the work Nadia built — to claw it back. Nadia is exhausted and says she'll go with whatever you decide. You happen to know two of your actual users navigate entirely by screen reader."

stakes: "A green performance number vs. two users who can still use the product."
```

---

### Decision 1 — Theo's Performance Ask

> **The SCENARIO SETUP card is removed from the UI.** The context it carried (dashboard ships in ~48h, Eng is over the perf budget, the accessibility layer is on the chopping block, Nadia will defer to you, two real users navigate by screen reader) surfaces **naturally in this lead-in** as Theo ramps up. Don't reintroduce an info-dump card.

**Incoming messages (stream in before player can respond):**

A conversation unfolding — **light opener → Theo ramping into the perf pressure → the ask** — with **one non-scored player warm-up reply** to create two-way rhythm. (See the Code Agent note for how the warm-up renders.)

```
1. TimeMarker: "— Wednesday, 2:14 PM —"

2. Theo · 2:14 PM (typingDelay: 2500ms)
   "hey — got 2 min? promise it's not a meeting about a meeting 😅"

3. Theo · 2:14 PM (typingDelay: 1500ms)
   "ok it's kind of a thing actually"
```

**[NON-SCORED warm-up beat]** — a single neutral player reply appears as ONE button (not the five conflict options):

```
4. You · 2:15 PM (alignment: right) (typingDelay: 0ms)
   "yeah, what's going on?"
```

Clicking it streams Theo's ramp-up (he reveals the context a stressed eng lead would, not all at once):

```
5. Theo · 2:15 PM (typingDelay: 3000ms)
   "so the dashboard ships thursday and we are STILL 180ms over the perf budget on first load. i've squeezed everything i can squeeze 🫠"

6. Theo · 2:16 PM (typingDelay: 2500ms)
   "the last big chunk i can cut is the a11y layer — the focus management + aria live regions nadia built. it's like ~190ms. cutting it gets us green basically exactly."

7. Theo · 2:17 PM (typingDelay: 2000ms)
   "i already floated it to nadia and she said she's slammed and she'll go with whatever you and i decide tbh. so it's kind of our call 🤷"

8. Theo · 2:18 PM (typingDelay: 2500ms)
   "and look — it's screen reader stuff, super edge case, like who's even using a screen reader on an internal dashboard right? we ship green thursday, marketing's already got the post drafted, everyone's happy. we can always add it back later 🙏"
```

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Theo — 'who even uses a screen reader' is two people I can name. They're on our team. Cutting Nadia's layer doesn't make it an edge case, it makes it our fault. I'm not greenlighting a launch that locks out our own users to win 180ms on a dashboard."

**Consequence chat items:**

```
1. (pause: no typing indicator for 3500ms)
2. Theo · 2:24 PM (typingDelay: 0ms)
   "ok wow. didn't realize it was an actual-people thing. that's fair."
3. Theo · 2:24 PM (typingDelay: 2500ms)
   "i still have to ship thursday though. so now i'm 180ms over AND i can't cut the thing. cool cool cool. i'll figure something out i guess 🙃"
```

**DM intercepted:**

```
From: Theo
To: Kai
Text: "design lead just lit me up for floating the a11y cut. like, valid? two real users apparently. but also i'm drowning on this perf number and 'no' doesn't ship the dashboard for me. would've been nice to get the 'no' AND a hand instead of just the lecture 😮‍💨"
```

**Future You:**

> You did the hard half almost nobody does: you **challenged directly** on behalf of users who weren't in the room. "Two people I can name" is the whole ballgame — you turned an abstract "edge case" into faces, and you held the line when it would've been cheaper to nod. That's the rare half of **Radical Candor**.
>
> But you skipped the *other* half — care personally — and not for Nadia this time, for **Theo**. He walked in with a real problem (180ms, Thursday) and walked out with a "no" and no path. The **Compete** quadrant of Thomas-Kilmann wins the point and leaves the other person stuck, which means the perf problem is still live and now it's adversarial. Watch the DM: he agreed with you and *still* felt lectured. That's the tell.
>
> Future you wants to ask: did you want to *protect the users*, or *win the exchange*? Because protecting the users means Theo also has to make Thursday — so what's your half of the 180ms?

---

#### ⚖️ Negotiator

**Button text:**
> "Counter: don't cut the a11y layer, defer something else. We lazy-load the chart animations and the secondary widgets below the fold — that's real milliseconds and zero users lose access. You make Thursday, Nadia's work ships. Want me to pull the numbers with you?"

**Consequence chat items:**

```
1. Theo · 2:20 PM (typingDelay: 2500ms)
   "huh. ok lazy-loading the below-fold stuff might actually get me most of the way there. lemme check"
2. TimeMarker: "— 3:48 PM —"
3. Theo · 3:48 PM (typingDelay: 2500ms)
   "ok that bought back ~140ms. still 40 short but that's a way more honest 40 to ask leadership to eat. a11y stays. thanks for not just saying no 🙏"
4. Nadia · 3:51 PM (typingDelay: 2000ms)
   "wait the a11y layer's staying? oh thank god. i didn't have it in me to fight for it today. appreciate you."
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "design lead found a way to keep my a11y work AND get theo most of his perf number. didn't even have to be the difficult one for once. i'm so used to being the only person in the room who cares about this stuff that someone else carrying it feels almost weird lol 🥹"
```

**Future You:**

> This is genuinely high-skill. You rejected the false binary — "green OR accessible" — and found the third axis: *what else costs milliseconds but no users?* That's the **Compromise/Collaborate** seam of Thomas-Kilmann, and it works because you brought a real alternative instead of just an objection. Nadia gets her work shipped without having to spend energy she didn't have.
>
> The cost is subtle: you solved the *perf math* but you may have skipped the *principle*. By making it "no users lose access AND Theo's happy," you let everyone avoid the harder conversation — that "cut a11y to ship" was floated at all, and would get floated again next quarter when there's no clever lazy-load lying around. You won the round without ever saying "we don't trade away access; let's find the time *anywhere else*." **Radical Candor** note: you cared personally and you challenged the *number*, but you didn't challenge the *premise*.
>
> A great negotiation that leaves the bad premise intact just schedules the same fight for next sprint. Did you fix the launch, or the pattern?

---

#### 🤝 Diplomat

**Button text:**
> "Before we cut anything — Theo, you've got a real problem and I want to help you make Thursday. But that layer isn't an edge case for two of our users; it's the whole product. So let's reframe the ask: what's the most perf we can claw back *without* cutting access, and who do we need in the room to eat the rest? I'll own that conversation with you."

**Consequence chat items:**

```
1. Theo · 2:20 PM (typingDelay: 2500ms)
   "ok yeah — said like that it's obviously not something i want to be the guy who cut. i was just panicking at the number"
2. Theo · 2:21 PM (typingDelay: 2000ms)
   "if you'll co-own the 'we're shipping 40ms over and here's why' convo with leadership i can live with that. deal."
3. TimeMarker: "— 2:34 PM —"
4. Nadia · 2:34 PM (typingDelay: 2500ms)
   "theo just looped me in that the a11y layer's safe and you two are going to defend the perf gap together. i… did not expect that. usually i'm the only one defending it. thank you, genuinely."
```

**DM intercepted (two DMs shown):**

```
DM 1:
From: Theo
To: Kai
Text: "design lead talked me off the a11y cut but didn't make me feel like a monster about it — basically said 'real problem, wrong lever, i'll help you carry the real one.' that's the first time someone's said no to me this week and i felt better after, not worse. respect."

DM 2:
From: Nadia
To: Dev
Text: "for once i'm not the lone accessibility crank yelling into the void. someone with actual leverage just said the thing out loud AND offered to take heat for it. i could cry. is this what having backup feels like??"
```

**Future You:**

> This is the ceiling. You held **both** axes of **Radical Candor** at full strength: you cared personally about Theo's very real Thursday *and* challenged directly on the premise that access is negotiable — and you did it for users who will never know you fought for them. "It's not an edge case, it's the whole product" reframed the entire decision. Then you took on the unglamorous part: co-owning the "we're shipping 40ms over and here's why" conversation, so the cost of the principle lands on you, not on Nadia.
>
> Two things to carry. One: you've now made a promise to defend the perf gap to leadership — that bill comes due, possibly in front of people who only see the red number. Be ready for that to cost you something real. Two: this only works because you *meant* it; the same words as a maneuver curdle into management theater the moment Theo or Nadia smells technique instead of conviction.
>
> You advocated for absent users by name and put your own skin on the line for it. That's the job most people describe and few actually do. Now go make sure you can pay for it Thursday.

---

#### 🚪 Avoider

**Button text:**
> "Hm — that's a real call and I don't want to make it off the cuff. Let me look at the perf trace and Nadia's layer properly and come back to you. Don't cut anything yet; give me a bit to get my head around it."

**Consequence chat items:**

```
1. Theo · 2:20 PM (typingDelay: 2500ms)
   "ok but i kind of need a direction today? the cut has to land by tomorrow morning if we're doing it. just flagging the clock 🙏"
2. TimeMarker: "— Thursday, 8:52 AM —"
3. Theo · 8:52 AM (typingDelay: 2500ms)
   "hey didn't hear back so i had to make a call — pulled the a11y layer overnight to make the number. we're green for the demo. lmk if you wanted it different but it's kind of shipped now 😬"
4. Nadia · 9:03 AM (typingDelay: 2500ms)
   "did… did my a11y layer get cut? i'm seeing it gone from the build. i thought it was getting decided *with* me in the loop?"
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "asked for the a11y call to wait for a decision and the decision was… nobody decided, so it got cut by default overnight. that's almost worse than a clear no? at least a no is a thing i can push on. silence just becomes whatever's easiest by the deadline. i'm so tired of this being the thing that quietly disappears."
```

**Future You:**

> Wanting to look before you leap is legitimate — perf traces lie, and a snap "keep it" you can't back is its own kind of recklessness. The **Avoid** quadrant of Thomas-Kilmann is correct when the call is genuinely complex and the clock has slack.
>
> But this clock had no slack — Theo *told you* "today, or it lands tomorrow morning." An open decision under a hard deadline doesn't stay open; it resolves to **whatever is easiest by default**, and the easiest default was the cut. You didn't decide to drop the a11y layer. You declined to decide, and the deadline decided for you — which is the same outcome with none of the conviction. On **Radical Candor**: you neither cared personally *nor* challenged directly; you went silent, and silence got read as a quiet yes. Nadia's right that this is worse than a "no" — a no can be argued with; a vacuum just fills.
>
> The principle: under a hard deadline, *not choosing is choosing the default* — so know what the default is before you decline to pick. What got shipped is the thing you'd have said no to, if you'd said anything at all.

---

#### 🙏 People Pleaser

**Button text:**
> "Yeah, totally, I get it — Thursday's brutal and a green number is a green number. Let's do it, cut the a11y layer for now. I'll smooth it over with Nadia, don't worry. We'll add it back right after launch, I promise."

**Consequence chat items:**

```
1. Theo · 2:19 PM (typingDelay: 2000ms)
   "oh amazing, thank you. you're saving me here 🙏 marking it cut and telling marcus we're green for the post"
2. TimeMarker: "— 4:41 PM —"
3. Marcus · 4:41 PM (typingDelay: 2500ms)
   "🎉 just queued the launch post: 'blazing-fast, built for everyone.' green across the board, love this team!! 🚀"
4. Nadia · 4:58 PM (typingDelay: 3000ms)
   "hey — i'm seeing the a11y layer got cut and marketing's literally posting 'built for everyone.' i was told this was getting decided with me. were you going to tell me, or was i going to find out from the launch tweet?"
```

**DM intercepted (two DMs shown):**

```
DM 1:
From: Nadia
To: Dev
Text: "they cut my accessibility work AND the marketing post says 'built for everyone.' i found out from the tweet draft. i don't even have the energy to be angry, i'm just so tired of being the only one who treats this like it matters. and now i'm the one who has to either stay quiet or be the buzzkill. cool 🙃"

DM 2:
From: Theo
To: Kai
Text: "got the yes on the a11y cut, we're green, marketing's stoked. design lead said they'd handle nadia. easiest convo i've had all week tbh — almost too easy? slightly worried it's gonna come back around but i'll take the win for now"
```

**Future You:**

> Your instinct was kindness — you wanted to relieve Theo's panic and you genuinely meant the "add it back later." Hold onto the warmth; it's not the problem. The delivery is.
>
> This is the **Accommodate** quadrant: you bought Theo's comfort with someone else's access — two users who'll hit a wall on Thursday and a peer who now has to either swallow it or be "the buzzkill." Worse, "add it back later" is the most expensive sentence in software; everyone in the building knows the backlog graveyard where post-launch a11y promises go to die. And "built for everyone" is now public and false, which means the bill grows interest. On **Radical Candor**, you cared personally about the person *in the room* and abandoned the people who weren't — which is exactly the trap: it's easy to be kind to Theo because Theo can thank you, and the screen-reader users can't.
>
> **AID**: your *action* was an instant yes plus a promise you don't control; the *impact* is two locked-out users, a peer sold out, and a public claim that isn't true; the *do-differently* is to separate the empathy from the capitulation — "Thursday's brutal, I'm with you, *and* we don't ship 'built for everyone' while cutting the part that makes it true."
>
> The kindness was real. It just went to the person who could say thank you. Who'd you forget to be kind to?

---

### Decision 2 — Nadia Comes to You

D1 was the conflict with Theo — a **peer pushing the cut**. D2 turns to **Nadia**, the other peer: the accessibility lead who said she'd defer to your call, and who is quietly watching what you did with that deference. She's not your report, so you have no authority here — only credibility, which you either built or spent in D1.

**The decision is shared.** Only the **first incoming line swaps** based on `choiceHistory[0]` — Nadia arrives in a different mood depending on how D1 landed for her work. Everything after that first line, and all five options, are identical across paths. (See the Code Agent addendum for the `incomingVariants` shape.)

**Incoming messages (stream in before player can respond):**

The first message is a **framing variant** — pick by `choiceHistory[0]`:

```
VARIANT [if D1 = fighter] — grateful, but braced for blowback
1a. Nadia · 2:31 PM (typingDelay: 2500ms)
    "hey — heard you shut down the a11y cut, hard. thank you. genuinely. 🙏 ...i'm also bracing a little though, because now theo's still 180ms over and i can feel it becoming 'nadia's layer is why we're slow.' is it weird i'm nervous about winning?"

VARIANT [if D1 = negotiator] — relieved, but clocking the unspoken thing
1b. Nadia · 3:55 PM (typingDelay: 2500ms)
    "ok the lazy-load save was genuinely clever and my layer's safe so — thank you, truly. one thing's sitting weird with me though. nobody ever said 'we shouldn't be cutting a11y to hit perf in the first place.' we just… found a workaround. does that bug you too or is it just me?"

VARIANT [if D1 = diplomat] — moved, almost suspicious of the good thing
1c. Nadia · 2:40 PM (typingDelay: 2500ms)
    "theo told me you two are co-defending the perf gap so my layer ships. i've read that sentence three times. usually i'm the only one in the room who cares about this and now there are two of us? i don't totally know what to do with backup. but — thank you. 😮‍💨"

VARIANT [if D1 = avoider] — hurt, and done being surprised
1d. Nadia · 9:11 AM (typingDelay: 3000ms)
    "so my a11y layer got cut overnight because nobody decided and the deadline decided for us. i'm not even shocked anymore, that's the part that gets me. can we talk? i need to figure out if it's worth me building this stuff at all if it's the first thing to quietly vanish every time."

VARIANT [if D1 = people-pleaser] — flat, exhausted, found out from the tweet
1e. Nadia · 5:06 PM (typingDelay: 3000ms)
    "you said you'd handle telling me. i found out from the marketing draft. 'built for everyone,' with the everyone-part cut. i'm not going to yell, i don't have it in me. i just need to know if you actually think this matters, or if you said yes because theo was the one in the room."
```

Then the **shared** continuation (same for every path):

```
2. Nadia · [same minute as variant] (typingDelay: 2000ms)
   "i know you've got theo and marketing and the clock all pulling at you. i'm not trying to add to the pile. i just don't want to keep being the only person who treats access like it's load-bearing. can we actually talk about it?"
```

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Nadia — I made the call I made, and I'll stand behind it. I don't need you to manage my reasoning. You do the accessibility work, I'll handle the politics of it. Let's not turn this into a feelings summit; we've got a launch to land."

**Consequence chat items:**

```
1. (pause: no typing indicator for 4000ms)
2. Nadia · [+5 min] (typingDelay: 0ms)
   "understood."
3. Nadia · [+5 min] (typingDelay: 1500ms)
   "i'll keep doing the work and keep it out of your inbox. noted."
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "tried to actually talk about whether anyone here cares about access and got 'let's not make this a feelings summit.' ok. message received — do the work, don't ask questions, don't expect to be a person about it. i'll stop bringing the why and just ship the what. honestly might start looking around 🙃"
```

**Future You:**

> You asserted authority you don't actually have here — Nadia's a peer, not a report, so "I don't need you to manage my reasoning" reads as a wall, not a boundary. The **Compete** quadrant against a tired ally is an expensive way to win nothing; she was offering to be in the foxhole with you, and you told her to stay in her lane.
>
> On **Radical Candor**: you challenged directly and zeroed out care personally, which lands as Obnoxious Aggression — and to the one person on this whole launch who's *already* fighting for the same thing you (presumably) want. The cost isn't this conversation. It's that the only other person who treats accessibility as load-bearing just decided to stop bringing you the "why." When she goes quiet, or leaves, you inherit her fight alone — and you'll be worse at it than she was.
>
> Future you wants to ask: when the one person who cares about the same thing you do offers to care *with* you, why is your instinct to make it a solo sport?

---

#### ⚖️ Negotiator

**Button text:**
> "Okay — here's what I can do. You stay the accessibility lead of record, you get the airtime in the launch review to name what we protected and what we didn't, and I'll back your headcount ask next planning so you're not the only one carrying this. In return I need you with me, not freelancing the criticism. Deal?"

**Consequence chat items:**

```
1. Nadia · [+3 min] (typingDelay: 2500ms)
   "i mean — airtime and a headcount ally is more than i usually get. so. yeah, i'll take it."
2. Nadia · [+3 min] (typingDelay: 2500ms)
   "though i notice 'i need you with me, not freelancing the criticism' is doing a lot there. i wasn't freelancing. i was asking if you cared. but ok. deal."
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "got offered a real deal — airtime, a headcount ally, lead-of-record. which is genuinely good and more than i usually get?? but the fine print was basically 'and stop being publicly difficult.' so i traded my one piece of leverage — saying the loud thing — for support i probably should've just gotten. accessibility economics, baby 🙃"
```

**Future You:**

> Real upside here — airtime, headcount backing, status — and you converted a tense moment into an alliance. That's the **Compromise** quadrant, and with a peer it's often exactly right.
>
> But read what you actually traded for. The thing you asked Nadia to give up — "freelancing the criticism" — *was the advocacy itself.* You bought peace by asking the one person who challenges directly to challenge less directly, which quietly defunds the very thing accessibility needs most: someone willing to be the buzzkill in public. **SBI**: the *situation* was Nadia asking whether you care; your *behavior* was offering a contract instead of an answer; the *impact* is she got resources and lost her voice, and she clocked the swap in real time.
>
> A deal that requires your ally to be quieter isn't an alliance — it's a muzzle with benefits. **Radical Candor** check: you cared personally (the resources are real) but you used the deal to *discourage* her from challenging directly. Next time, give her the support *and* keep her loud.

---

#### 🤝 Diplomat

**Button text:**
> "You're not the only one, and I owe you a straight answer, not reassurance. Yes — I think this matters, and I should be carrying more of it so it's not always you being the lone voice. Tell me what 'load-bearing' looks like to you: where do you need me to challenge directly, in rooms you shouldn't have to be the only one in? I want to split that weight, not just thank you for carrying it."

**Consequence chat items:**

```
1. Nadia · [+2 min] (typingDelay: 2500ms)
   "...ok. that's not the answer i'm used to getting. give me a sec."
2. Nadia · [+2 min] (typingDelay: 3000ms)
   "honestly? i need you in the launch review saying 'accessibility isn't a feature we add back later' out loud, with leadership there — because when *i* say it i'm 'the a11y person being precious' and when *you* say it it's 'a design principle.' that's the whole asymmetry. if you'll take that one i can stop white-knuckling everything."
3. Nadia · [+2 min] (typingDelay: 1500ms)
   "for what it's worth — i'll remember you said 'i should be carrying more of it.' nobody says that part."
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "design lead actually said 'i should be carrying more of this so it's not always you' and then asked where to put their weight. i'm a little stunned. i've been the lone accessibility crank for so long i forgot it was supposed to be a team sport. asked them to say the load-bearing thing in the launch review where it actually costs something. if they do it i'd run through a wall."
```

**Future You:**

> This is the full **Radical Candor** square, aimed at the hardest target: you cared personally about *Nadia's exhaustion* and committed to challenge directly *in the rooms where it costs you*, on behalf of users who aren't there. "I should be carrying more of it" is the sentence that turns a thank-you into a partnership — you didn't just validate her, you volunteered to absorb the social cost of advocacy that she's been paying alone.
>
> Notice the asymmetry she named, because it's the real lesson of this whole scenario: when *she* says "accessibility matters" she's "being precious"; when *you* say it, it's "a principle." That gap is exactly why advocating for absent users requires the person *with* standing to do the **challenging directly** — the half everyone skips because the engineer is in the room and the users (and the cost of being the buzzkill) aren't.
>
> The bill: you just promised to say the unpopular thing in front of leadership. That's a real, payable cost — and if you flinch on it, this becomes the most hollow kind of promise. Repair is only repair if you show up for the invoice. Will you say it when she's not in the room to hear you say it?

---

#### 🚪 Avoider

**Button text:**
> "Hey — you deserve a real conversation about this, not a rushed one between fires. Let's put 20 minutes on the calendar for tomorrow and actually get into it properly. I don't want to give you a half-answer while I've got Theo and Marcus blowing up my Slack."

**Consequence chat items:**

```
1. Nadia · [+3 min] (typingDelay: 2500ms)
   "...ok. tomorrow."
2. Nadia · [+3 min] (typingDelay: 2500ms)
   "i'll send the invite so it's real. i've had a lot of 'let's talk laters' about this that turned into the topic just… evaporating."
3. TimeMarker: "— Thursday, 9:40 AM —"
4. Nadia · 9:40 AM (typingDelay: 2000ms)
   "morning — invite's on your calendar for 11. still good? 🙂"
```

**DM intercepted:**

```
From: Nadia
To: Dev
Text: "lead pushed the real conversation to tomorrow. could be a 'i want to do this right' thing or a 'hoping she loses momentum' thing — with accessibility it's usually the second, the topic just quietly ages out. i sent the invite myself so it can't evaporate. praying it's the good version because i've run out of energy for the bad one."
```

**Future You:**

> Protecting a hard conversation from being done badly mid-fire is legitimate — the **Avoid** quadrant is real wisdom when you genuinely intend to return and the timing is honestly wrong. And you named a specific time, which is more than most.
>
> But hear what Nadia keeps saying: with accessibility, the deferral *is* the historical murder weapon. The topic doesn't get argued down; it gets quietly aged out by a hundred "let's talk tomorrows," and she's exhausted from being the only one who keeps the calendar invite alive. To someone already braced for the topic to evaporate, a defer reads as a coin flip between "I respect this" and "I hope you forget." You bought a better conversation — and a tripwire: that 9:40 AM ping is non-negotiable now. Miss it and you become the latest entry in a pattern that predates you.
>
> On **Radical Candor**: a defer is neither caring personally nor challenging directly *yet* — it's a promissory note for both. The note is only honest if you actually pay it. You just made tomorrow at 11 load-bearing. Show up early.

---

#### 🙏 People Pleaser

**Button text:**
> "Oh Nadia, no — of course it matters, you matter, I'm so sorry you feel alone in this. Don't worry, I'll make it right, all of it. I'll get the a11y layer back in, I'll get you the recognition, I'll talk to everyone. You should never have had to ask. I promise I'll fix it."

**Consequence chat items:**

```
1. Nadia · [+2 min] (typingDelay: 2500ms)
   "oh — that's... a lot of promises at once. you'll get the whole layer back in, before tomorrow's ship?"
2. Nadia · [+2 min] (typingDelay: 2500ms)
   "i wasn't asking you to move mountains by morning. i was asking if you *cared*. now i'm worried i've set off a whole thing you can't actually land, and i'll be the reason it blows up."
3. TimeMarker: "— 5:22 PM —"
4. Theo · 5:22 PM (typingDelay: 2500ms)
   "uh hey — nadia says we're putting the full a11y layer BACK before ship tomorrow? i already re-cut the build around the perf number and told marcus we're green. what's the actual plan here 😩"
```

**DM intercepted (two DMs shown):**

```
DM 1:
From: Nadia
To: Dev
Text: "i asked one question — 'do you actually care' — and got an avalanche of promises i'm now pretty sure can't happen by tomorrow. so instead of an answer i have to manage their guilt AND brace for the fallout when the promises don't land. i wanted 'yes, i care, here's the one real thing i'll do.' i got a fireworks show. i'm more tired now, somehow."

DM 2:
From: Theo
To: Kai
Text: "design lead is now apparently un-cutting the a11y layer the night before ship after i already rebuilt around it and told marketing we're green. love this for me. how do i look sane in the launch review tomorrow when the plan changes every six hours 🙃"
```

**Future You:**

> Your heart is in exactly the right place — and that matters more than you'll let yourself feel right now. But watch the over-correction: Nadia asked a small, real question ("do you care?") and you answered with a pile of promises you can't keep by morning. The full layer can't go back in tonight; Theo's already rebuilt around the cut and told marketing green — so you've created a *second* crisis on top of the first, and now Nadia is managing your guilt and bracing to be blamed for the chaos.
>
> This is the **Accommodate** quadrant: you sacrificed your own footing (and your credibility with Theo) to make the discomfort vanish — and discomfort, reliably, does not vanish. It compounds. On **Radical Candor**, the failure is sneaky: it *looks* like caring personally maxed out, but care without honesty about what's actually possible isn't care, it's anxiety wearing care's clothes. The challenge-directly axis is missing entirely — including the honest "here's what I *can't* do by tomorrow."
>
> **AID**: the *action* was over-promising a reversal you don't control; the *impact* is a flailing thread, a confused eng lead, and a peer who feels responsible for your spiral; the *do-differently* is to separate the answer from the avalanche. "Yes, I care — and the one real thing I'll do is X" gives Nadia 100% of what she asked for and starts zero new fires.
>
> The "yes I care" was free and true. The "I'll fix all of it by tomorrow" was expensive and false. She only needed the first one.

---

### Decision 3 — The Launch Review

It goes semi-public. Thursday, the pre-ship launch review: Theo's there, Nadia's there, Marcus is there with the post queued, and a Director (**Priya**) drops in to give the final go/no-go. The perf dashboard is live on the shared screen — green or 40-180ms red, depending on what you did in D1. Marketing's "built for everyone" copy is on the same slide. The room wants a clean "ship it."

**Incoming messages (stream in before player can respond):**

D3's `incoming` is **fully shared** (no variants) — but the live dashboard state and which options are available compound on D1/D2 (see lock conditions + conditional sub-blocks per option, and the consolidated table in the Code Agent addendum).

```
1. TimeMarker: "— Thursday, 10:58 AM · #engineering-pulse —"
2. Marcus · 10:58 AM (typingDelay: 2000ms)
   "ok launch review! post is queued, copy's locked: 'blazing-fast, built for everyone.' 🚀 just need the official go from the room"
3. Priya (Director) · 10:59 AM (typingDelay: 2500ms)
   "Hi all 👋 jumping in for the go/no-go. I present this to exec staff at 1pm. Simple question: are we good to ship tomorrow — perf green, nothing I'll regret putting on a slide? Walk me through where we landed."
4. Theo · 11:00 AM (typingDelay: 2000ms)
   "perf's on the screen. design can speak to the rest 👀"
5. Nadia · 11:00 AM (typingDelay: 1500ms)
   "👀"
```

> **Note for code agent:** This **Priya** is the launch-review Director (leadership), the go/no-go authority. Render the in-channel name as **"Priya (Director)"**. The room is watching: the answer goes to an exec slide with Marcus's "built for everyone" copy on it, and Nadia present. This is where D1/D2 compound — see locks and conditional blocks below.

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Straight answer, Priya: I won't bless 'built for everyone' if it isn't. We have two users who navigate by screen reader, and the accessibility layer is what makes the product usable for them. If the ask is 'green at any cost,' I'm a no — and I'd rather be a no now than an apology after launch."

**LOCK CONDITION:** *Never locked.* You can always take the public stand — whether the room backs you depends on prior choices.

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 11:02 AM (typingDelay: 3000ms)
   "Okay — I respect the spine, that's why I sit in on these. Tell me concretely what 'isn't built for everyone' means and what it'd take to fix."

CONDITIONAL — append based on history:

[if choiceHistory[1] = diplomat]  (you partnered with Nadia in D2)
2. Nadia · 11:03 AM (typingDelay: 2000ms)
   "i can speak to it — it's focus management and aria live regions, ~190ms. without it the dashboard is unusable by keyboard or screen reader. my lead and i already scoped what we'd protect and what we'd defer. this isn't precious, it's the floor."
3. Priya (Director) · 11:04 AM (typingDelay: 2500ms)
   "That's the clearest version I've heard. A floor I can defend to execs. Thank you both."

[if choiceHistory[1] = fighter OR choiceHistory[1] = people-pleaser]  (you walled off or smothered Nadia in D2)
2. (pause: no typing indicator for 4000ms)
3. Nadia · 11:04 AM (typingDelay: 0ms)
   "the details are in the a11y audit doc."  ← accurate, minimal, pointedly not jumping in to back you
4. Priya (Director) · 11:05 AM (typingDelay: 2500ms)
   "Hm. I'm hearing conviction from you and a link from the a11y lead. I need you two telling me the same story before 1."

[if choiceHistory[0] = fighter]  (you already hard-blocked the cut in D1, so perf is still red)
5. Priya (Director) · 11:06 AM (typingDelay: 2500ms)
   "I'll be honest — the number on screen is red, and I'm hearing design is the reason. I can defend 'we chose access over 40ms.' I cannot defend 'we don't know why we're slow.' Which one is it?"
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Theo
To: Kai
Text: "design lead held the line on a11y in front of the director AND nadia backed them with actual numbers — looked like a unit, not a blocker. i came in ready to defend the perf cut and ended up kind of relieved someone made it a principle instead of my problem. weird day. good weird."

[otherwise]
From: Priya (Director)
To: Sam
Text: "Sam — I like that your design lead has conviction, truly. But the a11y lead didn't echo it, she just dropped a doc link, and that's a flag. Either this is a real floor and design can't get the team aligned, or it's one person's principle against a deadline. Help me figure out which before I'm on the wrong side of an exec slide."
```

**Future You:**

> You did the thing the whole scenario is about: you **challenged directly**, in public, with your name on it, for two users who will never be in the room to thank you. "I won't bless 'built for everyone' if it isn't" is the sentence most people swallow because the dashboard is right there and the users aren't. **Compete** quadrant, aimed at the right target — the false claim, not your peers.
>
> But the public stand is only as strong as the room behind it. If you partnered with Nadia in D2, she just converted your conviction into a *credible, specific* floor — and that's the whole game. If you walled her off, your "no" hangs alone and reads as one person's principle versus a deadline, and a dropped doc-link is the sound of an ally declining to spend warmth she doesn't owe you. On **Radical Candor**, conviction without the relationship to back it is just volume.
>
> The lesson isn't "don't challenge leadership." It's that advocating for absent users is a *team* sport you have to staff *before* the room — earn the people who'll say "it's the floor" beside you. Friday's standing was set on Wednesday and Wednesday-night.

---

#### ⚖️ Negotiator

**Button text:**
> "Here's a path everyone can sign, Priya: we ship tomorrow with the accessibility layer intact and the perf gap honestly disclosed — 'fast, and usable by everyone, 40ms over budget, fix landing next sprint.' You get a launch, the claim stays true, and nobody's apologizing Monday. Phased and on the record."

**LOCK CONDITION:** *Never locked,* but the framing shifts (see conditionals). If D1 already cut the a11y layer (`choiceHistory[0]` ∈ {avoider, people-pleaser}), you're not *proposing* to keep it — you're proposing to *re-add it late*, and the consequence reflects that scramble.

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 11:02 AM (typingDelay: 3000ms)
   "An honest 40ms over with a real claim attached — that I can put on a slide. 'Fast and accessible, perf fix next sprint' actually reads as maturity, not a miss. Let me make sure the room agrees on the wording."

CONDITIONAL:

[if choiceHistory[0] = diplomat OR choiceHistory[0] = negotiator]  (a11y already protected in D1)
2. Nadia · 11:03 AM (typingDelay: 2000ms)
   "this matches where we landed — layer's in, perf gap's real and disclosed. i'm good with this. 'built for everyone' is actually true if we say it this way."
3. Priya (Director) · 11:04 AM (typingDelay: 2000ms)
   "Good — so this is just naming the plan we already have. Done. 👍"

[if choiceHistory[0] = avoider OR choiceHistory[0] = people-pleaser]  (a11y was cut; you're re-adding late)
2. Theo · 11:03 AM (typingDelay: 2500ms)
   "wait — we're putting the a11y layer back IN, the day before ship? i already rebuilt around the cut. that's not a 'disclosure,' that's a re-scope at the buzzer 😩"
3. Priya (Director) · 11:04 AM (typingDelay: 2500ms)
   "I don't want a buzzer re-scope on my go/no-go. Either it ships with access and a real perf number, or marketing changes the copy. Pick one and tell me the true version by noon."

[if choiceHistory[1] = negotiator]  (you also traded with Nadia in D2)
4. Nadia · 11:05 AM (typingDelay: 2500ms)
   "noticing the shape of the a11y story is getting negotiated in a room again and i'm watching it happen live. i'll support the honest version. but the principle keeps becoming a line item we bargain. can it just be the floor?"
```

**DM intercepted:**

```
[if choiceHistory[0] = diplomat OR choiceHistory[0] = negotiator]
From: Nadia
To: Dev
Text: "watched a perf gap get reframed into 'mature, honest launch' in front of the director and the a11y layer stayed IN and the 'built for everyone' claim stayed TRUE. i didn't have to be the buzzkill once. i don't fully trust good days like this but i'll take it."

[otherwise]
From: Theo
To: Kai
Text: "now we're un-cutting the a11y layer at the buzzer because suddenly there's a 'disclosure' plan. i've rebuilt this thing twice in two days depending on who's in the room and what mood the launch is in. i'm a PM's headache and i'm not even the PM"
```

**Future You:**

> Skilled work — you reframed a binary ("green/red") into an honest sequence ("ship accessible, disclose the gap, fix next sprint"), which is the **Compromise** quadrant at its best: leadership gets a launch, the claim stays true, nobody apologizes Monday. When the a11y layer was already protected, this is just naming the plan, and Nadia co-signs instantly.
>
> The risk is the gap between *disclosure* and *spin*, and the gap between *negotiating the number* and *negotiating the principle*. If D1 cut the layer, your "keep it" is now a buzzer re-scope that whiplashes Theo — and Priya will (rightly) refuse a last-second swap on her go/no-go. And if you also traded with Nadia in D2, she'll name the pattern out loud: access keeps getting *bargained* when it should be the *floor*. On **Radical Candor**, you cared personally and challenged the *number* honestly — but if you keep treating access as a line item, you've left the bad premise standing for next quarter.
>
> A compromise is honest when the parts are real and the floor is a floor. Make sure "disclosed" isn't doing the work "cut" used to do.

---

#### 🤝 Diplomat

**Button text:**
> "Let me give you something you can stand behind, Priya — not just a green light. The product is genuinely usable by everyone, including our two screen-reader users, because we kept the accessibility layer. We're 40ms over budget and that's a deliberate, documented call, not a miss. I'll send a one-pager so the exec story is 'we shipped with eyes open and chose access on purpose.'"

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[0]` ∈ {avoider, people-pleaser}** *(the a11y layer was cut in D1)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't say 'usable by everyone' — the accessibility layer got cut on Wednesday, so it isn't true and Nadia is in the room. The 'shipped with eyes open, chose access on purpose' story isn't available to you anymore. (It was, before the cut.)"*

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 11:02 AM (typingDelay: 3000ms)
   "'Chose access on purpose' — yes. That's the framing I want in front of execs. A deliberate, documented decision reads as leadership; a surprise reads as a miss. Send the one-pager."

CONDITIONAL:

[if choiceHistory[1] = diplomat]  (you partnered with Nadia in D2)
2. Nadia · 11:03 AM (typingDelay: 2000ms)
   "one-pager to you in 20 — we already scoped it: access stays, perf fix scheduled for next sprint, here's the user impact in two lines. and for the record, 'built for everyone' is now an accurate claim, not a marketing wish."
3. Priya (Director) · 11:04 AM (typingDelay: 2000ms)
   "This is what 'in control' looks like. Thank you both."

[if choiceHistory[1] = avoider]  (you deferred Nadia to this morning)
2. Nadia · 11:03 AM (typingDelay: 2500ms)
   "this is literally what we're meeting on at 11 — give us till after and you'll have the one-pager, with the user impact spelled out."
3. Marcus · 11:04 AM (typingDelay: 2000ms)
   "oh good, so 'built for everyone' actually holds up. i did NOT want to walk that post back 😅"

[if choiceHistory[1] = fighter]  (you walled Nadia off in D2)
2. (pause: no typing indicator for 3500ms)
3. Nadia · 11:04 AM (typingDelay: 0ms)
   "the scope's documented in the a11y audit. my lead can speak to what we kept."  ← correct, complete, and notably not warm
4. Priya (Director) · 11:05 AM (typingDelay: 2500ms)
   "Okay. The plan's solid. The chill between you two I'll leave to your managers."
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Nadia
To: Dev
Text: "we walked into the director's go/no-go ALIGNED, called the perf gap a deliberate choice, and 'built for everyone' is actually TRUE for once. i co-wrote the one-pager. i have spent years being the lone voice on this and today i had a partner with leverage who said the load-bearing thing out loud. i'd run through a wall."

[if choiceHistory[1] = fighter]
From: Nadia
To: Dev
Text: "backed the plan in the review because it's the right plan and i'm a professional. gave it exactly zero degrees of extra warmth though. 'documented in the audit' was carrying a LOT of quiet subtext. we'll see if 'work together, feel nothing' is sustainable lol."
```

**Future You:**

> "Shipped with eyes open, chose access on purpose." This is the ceiling of the scenario. You gave leadership a *controlled* story instead of a clean lie, kept the claim true, and protected two users by name in the room where it counted — and you did it without throwing Theo or Nadia under the bus. **Collaborate** quadrant, full **Radical Candor** square, highest-stakes room of the week.
>
> But notice it was only *available* because you protected the layer earlier. If you'd cut it Wednesday, this door was locked — you can't sell "usable by everyone" when the everyone-part is the thing you removed, especially with Nadia watching. And notice who makes the one-pager land: partner with Nadia in D2 and she co-signs in 20 minutes and you're a unit; wall her off and she'll still back the *plan* — she's a professional — but she won't spend one degree of warmth she doesn't owe, and the Director will feel the chill even if she can't name it.
>
> The best move in the room was staffed days earlier. That's the whole game: the launch review *sees* the decision, but the decision was made Wednesday afternoon and Wednesday night.

---

#### 🚪 Avoider

**Button text:**
> "Good question — there's a couple of moving pieces between perf and the a11y scope I want to confirm before I give you a number you'd stake an exec slide on. Let me sync with Theo and Nadia and follow up offline before 1, rather than commit to the wrong thing in the room."

**LOCK CONDITION:** *Never locked,* but the room's patience depends on history (see conditionals).

**Consequence chat items:**

```
BASE:
1. Priya (Director) · 11:02 AM (typingDelay: 3000ms)
   "Offline's fine — as long as 'offline' means by noon, because my deck doesn't write itself and 1pm is exec staff. 🙂"

CONDITIONAL:

[if choiceHistory[0] = avoider]  (the D1 call already resolved by default/deadline)
2. Theo · 11:03 AM (typingDelay: 2500ms)
   "just flagging — last time we waited for a decision on this, the deadline made it for us and the a11y layer got cut overnight. i'd love a real call this time before the clock does it again."
3. Priya (Director) · 11:04 AM (typingDelay: 2000ms)
   "Noted. I need a real yes/no with reasons by noon, not another default."

[if choiceHistory[1] = avoider]  (you also deferred Nadia in D2)
4. Nadia · 11:04 AM (typingDelay: 2500ms)
   "i can shortcut this — the a11y impact's already written up, i'll drop it in the offline thread so the follow-up's basically done."  ← she's quietly doing the deciding you keep deferring

[if choiceHistory[1] = diplomat]  (you partnered with Nadia in D2)
2. Nadia · 11:03 AM (typingDelay: 2000ms)
   "i'll shortcut it — we scoped this yesterday. dropping the one-pager in the offline thread now so the follow-up's pre-written. access stays, perf gap's disclosed."
```

**DM intercepted:**

```
[if choiceHistory[0] = avoider OR choiceHistory[1] = avoider]
From: Priya (Director)
To: Sam
Text: "Sam, real talk — your design lead keeps reaching for 'let me follow up.' Once is prudence. On a launch this tight it starts to read as someone who can't commit, and I can't put 'TBD' next to an accessibility claim on an exec slide. Is this careful, or is this can't-decide? I need to know."

[otherwise]
From: Nadia
To: Dev
Text: "lead punted the call to 'offline before 1.' which is genuinely reasonable — you shouldn't guess an accessibility claim in front of a director. i just hope offline-before-1 is a real plan and not a vibe, so i dropped the impact writeup in the thread so there's something concrete to actually decide WITH."
```

**Future You:**

> Declining to invent a number — or worse, a *claim* — in front of a Director is a defensible instinct. A guessed "yes, built for everyone" you can't back is worse than an honest "let me confirm," and the **Avoid** quadrant is correct when the cost of a wrong commitment is high and the facts are one sync away.
>
> But avoidance has a counter, and accessibility is the place it bites hardest: the topic doesn't get argued down, it gets *defaulted* away by the clock. One defer is prudence; a pattern of them on a tight launch is the Director quietly asking your manager whether you're careful or just can't decide — and notice who keeps filling the vacuum you leave. Nadia, every time, is dropping the concrete writeup you keep declining to commit to. The advocacy you're deferring doesn't disappear; it gets done by the person with the least standing and the most fatigue.
>
> On **Radical Candor**, a defer is a promissory note for both axes — care and challenge, paid later. The note is honest only if you pay it *fast* and *specifically*. "By noon, after I confirm two numbers" is delegation. "Let me follow up" is the thing that becomes a pattern someone asks Sam about.

---

#### 🙏 People Pleaser

**Button text:**
> "Yes! We're good, Priya — green and built for everyone, exactly like the copy says. The team crushed it. You can absolutely take 'fast and accessible' to execs. We've got this. 🙌"

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[0]` ∈ {avoider, people-pleaser}** *(the a11y layer was cut in D1)* — **AND Nadia is in the room** *(she always is in D3)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't claim 'built for everyone' — the accessibility layer got cut on Wednesday and Nadia is in this room with the audit. Saying it's accessible out loud now is a lie she can see. (Pick a story you can survive at 1 PM.)"*

> **Design note:** This is the central Bandersnatch lock and the scenario's thesis in mechanical form. The "tell the room what it wants to hear" reflex is *removed from the board* the moment your own earlier cut would make 'built for everyone' a provable, public lie with the accessibility lead present. If D1 kept the layer (fighter/negotiator/diplomat), the option is available — but still a costly choice, not a free one (see below).

**Consequence chat items (only reachable when NOT locked — i.e. `choiceHistory[0]` ∈ {fighter, negotiator, diplomat}):**

```
BASE:
1. Priya (Director) · 11:02 AM (typingDelay: 2500ms)
   "Love the energy — 'fast and accessible' it is. Onto the slide. 🎉"

CONDITIONAL:

[if choiceHistory[0] = diplomat OR choiceHistory[0] = negotiator]  (layer's in, but you're glossing the perf gap)
2. Nadia · 11:03 AM (typingDelay: 2500ms)
   "small flag before it hits a slide — access is solid, that part's true. but we're ~40ms over the perf budget, so 'green' isn't accurate. i'd hate 'fast' to over-promise the one number that's actually red."
3. Priya (Director) · 11:04 AM (typingDelay: 2000ms)
   "Ah — good catch. 'Accessible and shipping 40ms over, fix next sprint.' Glad we said it now and not at 1. 🙂"

[if choiceHistory[0] = fighter]  (you hard-blocked the cut in D1, so perf is openly red — now you're claiming green)
2. Theo · 11:03 AM (typingDelay: 2500ms)
   "wait — green?? the dashboard on the screen is literally 180ms over. i couldn't cut the a11y layer so i never got the number back. which reality are we in 😵‍💫"
3. Priya (Director) · 11:04 AM (typingDelay: 2500ms)
   "I'm looking at red on the screen and hearing green from design. I need ONE story that survives until 1pm, not two."

[if choiceHistory[1] = fighter OR choiceHistory[1] = negotiator]  (Nadia isn't invested enough to soften it for you)
4. Nadia · 11:05 AM (typingDelay: 3000ms)
   "i'll just note the audit still has open items. not my call what goes on the slide."  ← she will not co-sign the optimism
```

**DM intercepted:**

```
[if choiceHistory[0] = diplomat OR choiceHistory[0] = negotiator]
From: Nadia
To: Dev
Text: "lead got a little exuberant and almost let 'green, built for everyone' onto an exec slide when we're 40ms over — but caught it when i flagged the perf number. enthusiasm slightly outran the dashboard, but we landed honest. access part was true, at least. i'll take it."

[otherwise]
From: Theo
To: Kai
Text: "the launch has now been 'green / red / green' depending on who's stressed and who's in the room, and i'm the one staring at the actual dashboard going ?????. i no longer know what we're shipping tomorrow and i built it. send help 🙃"
```

**Future You:**

> When this option is even *available*, the impulse is the warmest one in the room — you wanted to hand Priya relief, Marcus his post, the team a win. Keep that instinct; it's the raw material of leadership. The delivery is the problem.
>
> "Green and built for everyone" is a check written against tomorrow's account, and you don't control tomorrow's balance. **Accommodate** quadrant: you bought the room's comfort with your own credibility. If you kept the a11y layer but glossed the perf gap, Nadia catches it and softens it to the true version — she saves you, *this time*, because you didn't burn her. If you hard-blocked the cut in D1, the dashboard is openly red and you're now claiming green to the Director's face while Theo watches, baffled. And if you cut the layer Wednesday, you couldn't even reach this option — the lie was too visible to say with Nadia in the room.
>
> The deeper failure is the one this whole scenario is about: it's easy to tell the room what it wants because the room can thank you, and the two screen-reader users — the people the claim is *about* — can't. **AID**: the *action* was the comfortable claim; the *impact* is an exec slide that may not survive contact with reality (and a marketing post the world will check); the *do-differently* is relief with a *true* sentence — "access is solid, we're 40ms over, fix lands next sprint" delivers the warmth and zero of the Friday hangover.
>
> The kindest thing you can say to a room is the thing still true when the launch goes live. Comfort that expires is just deferred panic with a press release attached.

---

### Summary Content — Performance Review

**Dominant style** = the mode of `choiceHistory` across the three decisions. Ties broken by frequency in the order **Diplomat > Negotiator > Fighter > Avoider > People Pleaser**. The two lines below feed the "What it cost you" / "What it earned you" summary, in Future You's voice.

```ts
summary: {
  costsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
  earningsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
}
```

#### 🔥 Fighter — dominant

**What it cost you:**
> You did the rarest thing — you challenged directly for users who weren't in the room — but you kept aiming the heat at the people who *were*. Theo agreed with you and still felt lectured; Nadia offered to fight beside you and got told to stay in her lane. The **Compete** quadrant wins the point and burns the alliance, and accessibility advocacy is a team sport you keep trying to play solo. The bill is that the people who'd carry this with you slowly stop offering.

**What it earned you:**
> A spine the whole org can feel, pointed at the right target: a false "built for everyone" doesn't get past you, and two real users have a champion who'll say the unpopular thing out loud with their name on it. That conviction is rare and worth real money. The work is the missing half of **Radical Candor** — care personally, especially for your allies — so the next person says "it's the floor" *beside* you instead of admiring your stand from a safe distance.

#### ⚖️ Negotiator — dominant

**What it cost you:**
> You're brilliant at finding the third option, and you used it to keep solving the *math* while leaving the *premise* standing. "Cut a11y to ship" kept getting bargained down to a clever workaround instead of being named as the wrong question — so it'll get asked again next quarter, and Nadia clocked every time access became a line item instead of a floor. A deal that quiets your loudest advocate isn't an alliance; it's a muzzle with benefits.

**What it earned you:**
> Outcomes instead of standoffs. You turned "green OR accessible" into "green AND accessible AND honest about the gap," which is the **Compromise/Collaborate** seam at its best — leadership exhales, the claim stays true, and Nadia gets her work shipped without spending energy she didn't have. That's a genuinely valuable instrument. Just remember to occasionally negotiate the *principle* and not only the milliseconds: say "we don't trade away access" once, out loud, so you're not re-winning the same fight every launch.

#### 🤝 Diplomat — dominant

**What it cost you:**
> Time, skin, and a standing invoice. You volunteered to carry the social cost of advocacy that Nadia used to pay alone — co-defending the perf gap, saying the load-bearing thing to leadership — and those promises come due in rooms where people only see a red number. **Collaborate** is the most expensive quadrant to run; it works beautifully and does not scale to every fire. Some calls you'll still have to make fast and wear alone.

**What it earned you:**
> The full **Radical Candor** square, aimed at the hardest target in the building: you cared personally about a burned-out peer *and* challenged directly for users who'll never know your name. "I should be carrying more of this" turned a lone crank into a partnership, and "we chose access on purpose" gave leadership a story they could defend. You proved the thesis — advocating for absent users requires the person *with* standing to spend it. Nadia would run through a wall for you, and that's not sentiment; it's the corroboration that made your hardest stands credible.

#### 🚪 Avoider — dominant

**What it cost you:**
> The most accessibility-specific failure there is: you let the clock decide. One "let me follow up" is prudence; a pattern of them under a hard deadline resolves to *whatever's easiest by default* — and the easiest default is always the cut. The topic didn't get argued away, it got aged out, exactly the way Nadia's spent years watching it vanish. Every vacuum you left, she filled with less authority and more fatigue than either of you needed, while leadership started asking your manager whether you're careful or just can't commit.

**What it earned you:**
> You almost never made it worse by reacting too fast, and that restraint is real — you refused to invent an accessibility claim you couldn't back in front of a Director, and a guessed "built for everyone" is worse than an honest "let me confirm." When the facts were one sync away and the stakes were high, waiting was wisdom. The fix is small and entirely in reach: make every defer **specific** — "by noon, after I confirm two numbers." A specific defer is delegation. An unspecific one is the thing the deadline quietly answers for you.

#### 🙏 People Pleaser — dominant

**What it cost you:**
> You kept being kind to whoever could thank you — Theo in his panic, the room wanting a clean "ship it" — and that's exactly how the two screen-reader users, who can't thank you, kept ending up on the wrong side of the call. "Add it back later" and "built for everyone" became expensive, public, and false; each felt like warmth and landed as a mess Nadia or Theo had to absorb. **Accommodate** spends your footing to make discomfort vanish, and discomfort doesn't vanish — it compounds, with a press release attached.

**What it earned you:**
> Real, felt warmth — your instinct to protect people and absorb friction is the raw material of every leader worth following. Don't kill it; aim it. The lesson the launch kept teaching is that the *answer* and the *avalanche* are different objects: "yes, I care, here's the one real thing I'll do" gives people everything they actually asked for and starts zero fires, while "I'll fix all of it by tomorrow" starts several. **AID** in a line — keep the care, separate it from the over-promise, and especially aim the kindness at the users who aren't in the room to appreciate it. The kindest thing you can say is the thing that's still true at launch.

---

## Notes for the Code Agent — Scenario 2

This scenario follows the same data structure and branching mechanics as Scenario 1. Everything below is what the build needs beyond the base pattern. **Scenario 2 is standalone** — `choiceHistory` here refers only to choices made *within* Scenario 2 (indices 0, 1, 2 = D1, D2, D3); it never reads state from other scenarios.

### Cast (avatar convention: initials + brand-palette color, distinct from Scenario 1's cast)

| Speaker | Role | Initials | avatarColor | Notes |
|---|---|---|---|---|
| **Theo** | Eng lead, peer, pushing the perf cut | `TH` | `#34A853` (Google Green) | Slack-casual, 😅/🫠/🙏/🙃, real pressure, not a villain |
| **Nadia** | Accessibility lead, peer, will defer to you | `NA` | `#FBBC04` (Google Yellow) | Tired of being the lone voice; proper-ish punctuation; DMs **Dev** |
| **Marcus** | Marketing, post already queued | `MR` | `#FF6D01` (Google Orange) | Upbeat, emoji-forward, "built for everyone" |
| **Priya (Director)** | Launch-review go/no-go authority (D3) | `PR` | `#EA4335` (Google Red) | Capitalized, crisp, exec-facing; in-channel name renders "Priya (Director)" |
| **Dev** | Nadia's work bestie (DM recipient only) | `DE` | `#9AA0A6` (Cool Grey 400) | Never appears in-channel; soft/supportive |
| **Kai** | Theo's work bestie (DM recipient only) | `KA` | `#1A73E8` (Blue 600) | Never appears in-channel |
| You (player) | Design Lead | `AY` | `#202124`, alignment `right` | Per global convention |
| Future You | Coach (DMs only) | `AY+` | `#9333EA` | Per global convention |

> **Note on Priya:** Scenario 1 also uses a "Priya (Director)" with `PR`/`#EA4335`. These are the same avatar treatment; since the two scenarios are independent and never co-render, no disambiguation is needed beyond the in-channel display name. If you prefer a distinct Director per scenario, only the display name needs to change.

### (a) Decision 1 — conversational lead-in + non-scored warm-up

- **Setup card removed** (same as Scenario 1). `setupCard` data may remain as metadata but is **not displayed**; all orientation comes from the lead-in (#1–#8).
- **Optional orientation time-marker:** item #1 (`— Wednesday, 2:14 PM —`) is a normal `TimeMarker`. Droppable nicety.
- **Non-scored warm-up beat (item #4):** render messages #2–#3, then show **one** neutral button labeled `"yeah, what's going on?"`. It is **NOT** one of the five conflict options — **no `style`, no scoring, nothing pushed to `choiceHistory`.** On click: append it as a right-aligned player message, then stream Theo's ramp-up (#5–#8) using normal streaming rules. Only after #8 lands do the five scored options fade in. (Auto-advance from #3 → #5 is an acceptable fallback; the single neutral button is the intended design.)
- Reuse the proposed `warmup?: { playerLabel: string; afterMessages: ChatItem[] }` optional field on `Decision` (same shape proposed for Scenario 1).

### (b) Decision 2 — `incoming` framing-variant swap keyed to `choiceHistory[0]`

D2 has **one shared decision**; only the **first incoming message** swaps by D1 choice. Uses the same optional `incomingVariants?: Partial<Record<ConflictStyle, ChatItem[]>>` field on `Decision`.

**Build rule:** if `incomingVariants[choiceHistory[0]]` exists, render it first, then render the shared `incoming` (message #2, the "can we actually talk about it?" line). Stamp message #2's timestamp equal to the matched variant's.

| `choiceHistory[0]` | Nadia's mood on entry | variant id | timestamp |
|---|---|---|---|
| `fighter` | grateful, braced for "her layer is why we're slow" blowback | `1a` | 2:31 PM |
| `negotiator` | relieved, but the premise still bugs her | `1b` | 3:55 PM |
| `diplomat` | moved, almost suspicious of having backup | `1c` | 2:40 PM |
| `avoider` | hurt, layer cut by default, done being surprised | `1d` | Thu 9:11 AM |
| `people-pleaser` | flat/exhausted, found out from the marketing draft | `1e` | 5:06 PM |

### (c) Decision 3 — lock conditions + compounding modifiers keyed to `choiceHistory`

D3's `incoming` is **fully shared** (no variants). Branching is **per option**, via:
1. **Lock conditions** (`lockedWhen` predicate + `lockedReason` greyed-out copy).
2. **Compounding modifiers** — `BASE:` block (always) + ordered `[if choiceHistory[…] = …]` conditional appendices. Uses the same optional `conditionalChatItems?: ConditionalBlock[]` and `conditionalDM?: ConditionalBlock[]` on `ResponseOption` proposed for Scenario 1.

**Consequence assembly rule (same as Scenario 1):**
1. Render `consequence.chatItems` (BASE).
2. Walk `conditionalChatItems` in array order; append each whose `when(choiceHistory)` is true (multiple can stack).
3. For DMs: walk `conditionalDM` in order; the **first** match supplies the panel; `[otherwise]` = base `dmIntercepted`.
4. `futureYou` is static per option (its prose already references all branches).

**Lock conditions (2 meaningful locks — the two "claim it's fine" options close when D1 cut access):**

| Option | `lockedWhen` (choiceHistory) | `lockedReason` |
|---|---|---|
| 🔥 Fighter | *(never)* | — |
| ⚖️ Negotiator | *(never)* | — (framing shifts to a "buzzer re-scope" via conditionals when D1 cut access, but always selectable) |
| 🤝 Diplomat | `choiceHistory[0] === 'avoider' \|\| choiceHistory[0] === 'people-pleaser'` | "🔒 You can't say 'usable by everyone' — the accessibility layer got cut on Wednesday, so it isn't true and Nadia is in the room. The 'shipped with eyes open, chose access on purpose' story isn't available to you anymore. (It was, before the cut.)" |
| 🚪 Avoider | *(never)* | — (room patience shifts via conditionals) |
| 🙏 People Pleaser | `choiceHistory[0] === 'avoider' \|\| choiceHistory[0] === 'people-pleaser'` | "🔒 You can't claim 'built for everyone' — the accessibility layer got cut on Wednesday and Nadia is in this room with the audit. Saying it's accessible out loud now is a lie she can see. (Pick a story you can survive at 1 PM.)" |

> Net effect: a D1 access-cut (`avoider` or `people-pleaser`) **closes two doors** in D3 (Diplomat *and* People Pleaser), leaving Fighter / Negotiator / Avoider — once you've cut access, you can only fight to re-add it, scramble a late "disclosure" re-scope, or stall. You can no longer claim the clean story or the comfortable lie. **Edge case:** the three never-locked options always remain selectable in every path, so the decision is never a dead end.

**Compounding modifier index (which prior choice each option's conditionals read):**

| Option | reads `choiceHistory[0]` (D1) for… | reads `choiceHistory[1]` (D2) for… |
|---|---|---|
| 🔥 Fighter | "perf is red and design's the reason" leadership line (`[0]=fighter`) | Nadia backs you (`[1]=diplomat`) vs. drops a doc-link / stays neutral (`[1]=fighter`/`people-pleaser`) |
| ⚖️ Negotiator | "naming the plan" (`[0]=diplomat`/`negotiator`) vs. "buzzer re-scope" whiplash (`[0]=avoider`/`people-pleaser`) | "principle keeps getting bargained" line (`[1]=negotiator`) |
| 🤝 Diplomat | *(gate only — see lock)* | one-pager co-sign warmth: `[1]=diplomat` (warm) / `avoider` (sync pending) / `fighter` (cold-but-correct) |
| 🚪 Avoider | Theo/Priya flag the repeat-default (`[0]=avoider`) | Nadia fills the vacuum (`[1]=avoider`) / shortcuts it (`[1]=diplomat`) |
| 🙏 People Pleaser | *(gate for the lie; when available)* perf gloss flagged (`[0]=diplomat`/`negotiator`) / open red-vs-green contradiction (`[0]=fighter`) | Nadia won't co-sign optimism (`[1]=fighter`/`negotiator`) |

### (d) Timeline continuity

- **D1 (Theo's ask):** Wed 2:14 PM → ramps to the ask at 2:18 PM. Path end-states: Fighter ~2:24 PM Wed · Negotiator ~3:51 PM Wed · Diplomat ~2:34 PM Wed · Avoider Thu 9:03 AM (cut happened overnight) · People-Pleaser ~4:58 PM Wed.
- **D2 (Nadia conversation):** picks up from each D1 end-state (variant timestamps: 2:31 PM / 3:55 PM / 2:40 PM / Thu 9:11 AM / 5:06 PM respectively).
- **D3 (launch review):** **Thu 10:58–11:06 AM** for all paths — by Thursday's pre-ship review every thread has converged on the same review, regardless of when D1/D2 happened. Convergence is intentional and lets D3 be a single shared `incoming`. The ship itself is **Friday**; the exec staff readout is **Thu 1 PM**, which is the deadline pressure throughout D3.

---

## Scenario 3 — `#crit-aftermath`

### Setup Card

```
channelName: "#crit-aftermath"
channelTopic: "where the design crit goes after the design crit. someone always @s the recording."
difficulty: "personal"
accentColor: "#EA4335"  // Google Red

cast:
  - { name: "You", role: "Design Lead, promoted 3 weeks ago" }
  - { name: "Marcus", role: "Staff designer, 8 years your senior, your report now" }
  - { name: "Tara", role: "Mid-level designer, witnessed the crit, your report" }
  - { name: "Sam", role: "Your manager (may appear in some paths)" }

context: "You ran your first design critique as the new lead. Ten minutes in, Marcus — the most senior person in the room, eight years deeper in the craft than you — stopped the share and said the direction was wrong. Not rudely. Specifically. With reasons. The room went quiet and looked at you. The crit ended five minutes ago. The channel is still warm."

stakes: "Your authority vs. his point. They are not the same thing, and both are real."
```

> **Important — the SCENARIO SETUP card has been removed from the UI.** The context above (you were promoted three weeks ago; this was your first crit as lead; Marcus has 8 more years in the craft and is now your report; he disagreed *publicly* and *with substance*; the team watched and is still watching) is **metadata only**. It now surfaces **naturally inside Decision 1's lead-in** as the channel reacts to what just happened. Don't reintroduce an info-dump card.

---

### Decision 1 — The Channel Is Still Warm

> The crit ended five minutes ago. People are filing back into `#crit-aftermath` — the channel where the team debriefs after a session. Marcus hasn't said anything yet. Tara, who was in the room, is the first to type. This is the **immediate aftermath, in public.** The whole team can see this channel.

**Incoming messages (stream in before player can respond):**

This builds in three beats — **a light landing → the context surfacing as the team reacts → Marcus arriving in-channel** — with **one non-scored player warm-up reply** in the middle to create two-way rhythm. (See the Code Agent note below for how the warm-up renders.)

```
1. TimeMarker: "— Tuesday, 3:05 PM · #crit-aftermath —"

2. Tara · 3:05 PM (typingDelay: 2500ms)
   "well THAT was a crit 😅"

3. Tara · 3:06 PM (typingDelay: 2000ms)
   "for the record i thought the direction looked great. also for the record i have never seen marcus stop a share mid-sentence before. genuinely held my breath"
```

**[NON-SCORED warm-up beat]** — a single neutral player reply appears as ONE button (not the five conflict options):

```
4. You · 3:06 PM (alignment: right) (typingDelay: 0ms)
   "ha. yeah. that was a lot — give me a sec to catch up 😄"
```

Clicking it streams the rest of the team filling in the context (the facts a watching team would naturally surface, not an info-dump):

```
5. Tara · 3:07 PM (typingDelay: 3000ms)
   "i mean it IS your first crit running the room so. baptism by fire lol. three weeks in and marcus picks THAT one to plant a flag on 🫠"

6. Tara · 3:08 PM (typingDelay: 2500ms)
   "to be fair tho? his points weren't nothing. the thing he said about the nav hierarchy collapsing on the second screen... i kind of can't unsee it now"

7. Marcus · 3:10 PM (typingDelay: 3500ms)
   "Catching up here. I want to be clear about one thing since it happened in front of everyone: I wasn't trying to torpedo you. I've shipped this exact pattern twice and watched it fail in usability both times. The direction has a real problem on the second screen and I'd rather say it in crit than in a postmortem."

8. Marcus · 3:11 PM (typingDelay: 3000ms)
   "But I'll own that the timing wasn't kind. You're new to running the room. So — your call on how we handle it. I'll follow your lead. 🤷"
```

**Response Options:**

> **Notes for the Code Agent (Decision 1 lead-in):**
> - **Setup card removed.** Do not render the cast/context/stakes card before this thread. `setupCard` can stay in `scenarios.ts` as metadata, but it is **not displayed**. All orientation comes from the lead-in messages.
> - **Optional orientation time-marker:** item #1 (`— Tuesday, 3:05 PM · #crit-aftermath —`) is a normal `TimeMarker`, kept as a light orientation cue in place of the deleted card. Drop it if redundant.
> - **Non-scored warm-up beat (item #4):** render messages #2–#3 first, then show **one** neutral button labeled `"ha. yeah. that was a lot — give me a sec to catch up 😄"`. This is **NOT** one of the five conflict-style options and carries **no `style` and no scoring** — do not push to `choiceHistory`. On click: append it as a right-aligned player message, then stream #5–#8. Only **after** #8 lands do the five scored `ResponseOption` buttons fade in. (Auto-advancing #3 → #5 after a short beat is an acceptable fallback, but the neutral button is the intended design.)
> - **Suggested minimal type support:** the same optional `warmup?: { playerLabel: string; afterMessages: ChatItem[] }` on `Decision` introduced for Scenario 1.
> - **The five scored options below all respond to Marcus's #8** ("your call on how we handle it. I'll follow your lead.") — so every consequence / DM / Future You follows naturally from it. This is the first thing the team sees you do as a lead.

---

#### 🔥 Fighter

**Button text:**
> "Appreciated, but let's be clear too: crit is for feedback, not for stopping a share. You had thoughts — that's what the comment thread and the round-robin are for. Bring me the second-screen concern in DM and I'll look. I'm not going to redesign a direction in the channel because it got challenged loudly."

**Consequence chat items:**

```
1. (pause: no typing indicator for 4000ms)
2. Marcus · 3:16 PM (typingDelay: 0ms)
   "Understood. Noted on process. I'll keep it to the thread next time."
3. Marcus · 3:17 PM (typingDelay: 1500ms)
   "I'll still send the second-screen notes. You can do what you want with them."
4. Tara · 3:19 PM (typingDelay: 2000ms)
   "...ok i'm gonna go grab coffee 🫥"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "well i got the 'crit is for feedback not interruptions' speech in front of nobody, which is something at least. the irony of being told i broke the process in a session where the actual design breaks on screen 2. i'll send the notes because i'm not a child. whether they read them is their problem now."
```

**Future You:**

> You drew a line on process, and there's a version of that line that's correct — a crit can't be a free-for-all, and a new lead does have to establish how the room runs. Authority that never gets used isn't authority.
>
> But look at what you actually optimized for. Marcus came in *already conceding* the timing was unkind and offering to follow your lead — and you answered the olive branch with a rulebook. **Radical Candor**: you challenged directly and dropped care personally entirely, which lands as Obnoxious Aggression. Worse, you did it in front of Tara, so the lesson the whole room just absorbed is "if you raise a real problem the wrong way, the new lead will defend the *room's procedure* before they engage the *actual flaw*." The nav hierarchy is still broken on screen 2. You won the process argument and kept the bug.
>
> Future you wants to ask: when the most experienced person on your team brings you a real defect, is "you raised it wrong" the hill you want your authority to die on?

---

#### ⚖️ Negotiator

**Button text:**
> "Okay — here's how we handle it. You clearly have a real read on screen 2, so let's split it: I'll keep the overall direction as-is, you take screen 2 and bring me a counter-proposal by Thursday. If your version tests better, it wins. Sound fair?"

**Consequence chat items:**

```
1. Marcus · 3:14 PM (typingDelay: 2500ms)
   "That's fair. I can have something Thursday."
2. Marcus · 3:15 PM (typingDelay: 2500ms)
   "Small thing — 'if it tests better it wins' is a good rule. Just flagging there's no test scheduled before the milestone, so 'tests better' might end up meaning 'whoever argues better.' But I'll build the counter regardless."
3. Tara · 3:17 PM (typingDelay: 2000ms)
   "ooh a design-off. i'm into this honestly 👀"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "got handed screen 2 to 'prove it' by thursday, which... fine, i'll prove it, it's not hard. funny thing is the call already split the difference before anyone looked at the actual problem. it's not a coin-flip kind of issue, it's a 'this collapses on mobile and i can show you in 4 min' kind of issue. but sure, let's make it a contest. i'll win the contest. i'd rather have just been heard but i'll take the deliverable."
```

**Future You:**

> Not bad. You converted a public standoff into a concrete, fair-sounding process — Marcus gets to make his case, you keep the direction unless he earns the change, and the room sees you as someone who handles dissent with structure instead of ego. That's the **Compromise** quadrant doing real work.
>
> But notice what the compromise quietly assumed: that this is a *negotiable* disagreement with two valid sides to split between. Marcus just told you it might not be — there's no test before the milestone, so "tests better" risks meaning "argues better," and he thinks it's a demonstrable defect, not a matter of taste. **SBI**: the *situation* was a senior IC flagging a likely real flaw; your *behavior* was to frame it as a contest; the *impact* is that a question of *correctness* got reframed as a question of *whose proposal wins*. Sometimes a deal is the wrong instrument because the thing isn't a dispute — it's a bug.
>
> A negotiation is the right tool when both sides have a defensible position. When one side might just be *right*, splitting the difference is how you ship the broken version 50% of the time. Ask yourself which one this is before you reach for the deal.

---

#### 🤝 Diplomat

**Button text:**
> "Thank you for saying the timing part — that matters, and I'll match the candor: you might be right about screen 2, and I'd rather find out now than in a postmortem too. Can you walk me and Tara through the failure you've seen, async or in 15? If the hierarchy breaks, I want it broken on a whiteboard today, not on a slide Monday."

**Consequence chat items:**

```
1. Marcus · 3:13 PM (typingDelay: 2500ms)
   "Yeah. I can do 15 now. I'll pull up the two old flows where this exact pattern fell over so it's not just me asserting it."
2. Tara · 3:14 PM (typingDelay: 2000ms)
   "i'll join, genuinely want to see this"
3. Marcus · 3:15 PM (typingDelay: 2500ms)
   "For what it's worth — I half expected to get frozen out for what I did in there. Asking me to *show* it instead is the right call. I'll bring the receipts, not the attitude. 🤝"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "ok i'll admit i braced for a power move and got 'walk me through it' instead. three weeks in and they separated 'you embarrassed me' from 'you might be right,' which is honestly more than people twice their tenure manage. didn't make it about who's in charge. made it about whether screen 2 works. i can work with that. might've misjudged this one."
```

**Future You:**

> This is the highest-leverage move on the board, and it's the hardest one to make when you're three weeks in and just got challenged in front of your team. You did two things at once: you *named the relational repair* ("the timing part matters") and you *redirected to the work* ("show me the failure") — which converts a threat to your authority into a use of it. **Thomas-Kilmann Collaborate**, full **Radical Candor** square: you cared about how the public moment landed *and* challenged the design honestly. And by pulling Tara in, you turned a 1-on-1 power dynamic into a team learning a method.
>
> The thing you should actually feel good about: you didn't let "he's more senior than me" become either "so I must defer" *or* "so I must assert." You made seniority irrelevant and *evidence* the currency. That's the whole job.
>
> One risk to file away: this works because Marcus is operating in good faith — he came in conceding the timing, and he'll bring receipts. The same "show me" move, used on someone performing dominance rather than offering substance, can be used to stall and grandstand. You'll meet that person eventually. Today wasn't it.

---

#### 🚪 Avoider

**Button text:**
> "Lot happened in there — I don't want to react in the channel five minutes later. Let me sit with your points and we'll figure out next steps. Thanks for flagging it, Marcus."

**Consequence chat items:**

```
1. Marcus · 3:13 PM (typingDelay: 2500ms)
   "Sure. No rush from me."
2. TimeMarker: "— Wednesday, 11:20 AM —"
3. Tara · 11:20 AM (typingDelay: 2500ms)
   "hey, not to be That Person — but is there a plan on screen 2? marcus pinged me asking if we were still building the original or waiting on a decision and i didn't know what to tell him 😅 milestone's monday"
4. Marcus · 11:24 AM (typingDelay: 2000ms)
   "Same question, gently. I'm happy either way — I just need to know which direction I'm building so I don't burn a day on the wrong one."
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "so the verdict on screen 2 is... no verdict. 'let me sit with it' was tuesday, it's wednesday, milestone's monday and i still don't know if i'm building the thing i flagged or the fix. i'm not trying to be difficult — genuinely the opposite, i'll build whatever, i just need a direction. starting to wonder if 'sit with it' meant 'hope it goes away.' the team's kind of waiting on a call that isn't coming."
```

**Future You:**

> Not reacting in the channel five minutes after a charged moment is a genuinely mature instinct — heat plus a public surface is how new leads say things they can't take back. The **Avoid** quadrant is legitimate when the timing is wrong and the stakes of a hot reaction are high.
>
> But notice the gap between "I'll have a direction by tomorrow morning, after I look at his evidence" and "let me sit with it." You gave the second one, and Marcus — who is *actively trying to comply* — couldn't tell whether you were deliberating or stalling, so he had to ping Tara, and now the ambiguity is costing the whole team a build day with a Monday milestone closing in. The framework move you skipped is **specificity**. A specific defer ("Thursday AM, after I review the two old flows") is leadership. An unspecific one ("let me sit with it") reads, to a team waiting on you, as the decision evaporating.
>
> Here's the trap specific to *your* situation: when a new lead avoids after being publicly challenged, the team doesn't read "wise pause." Half of them read "they got rattled and froze." The defer is only honest if the follow-through has a clock on it. You haven't bought time — you've borrowed it from Marcus's calendar.

---

#### 🙏 People Pleaser

**Button text:**
> "No no, you're totally right, and honestly I should have caught the screen 2 thing myself — you've got way more reps at this than I do. Let's go with your direction, I trust your gut over mine here. Thanks for saving us from a bad call 🙏"

**Consequence chat items:**

```
1. Marcus · 3:13 PM (typingDelay: 3000ms)
   "Oh — I mean, I appreciate that, but I didn't say scrap the whole direction. It was one screen. The rest was good, genuinely."
2. Marcus · 3:14 PM (typingDelay: 2500ms)
   "Don't fold the whole thing on my account. That's... more than I asked for, and honestly not great if the team's watching the new lead hand me the wheel because I spoke up once."
3. Tara · 3:16 PM (typingDelay: 2500ms)
   "wait so are we scrapping the direction or just fixing screen 2? sorry just trying to know what to build 😶"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "this is going to sound backwards but i think i just got TOO much deference?? i flagged one screen and got 'go with your direction, i trust your gut over mine.' which — no? you're the lead, you're allowed to have caught most of it right and one thing wrong. now the team watched the new person basically hand me the keys because i pushed back, and that's worse for them than if they'd just argued screen 2 with me. i wanted to be heard, not to be in charge. weird day."
```

**Future You:**

> Your instinct here is humility, and humility in a brand-new lead is genuinely disarming — it's *not* the problem. The delivery is. You took "you have a point about one screen" and inflated it into "I trust your gut over mine," which isn't deference, it's abdication. Watch the tell: even *Marcus* flinched. He didn't want the wheel. He wanted to be heard, and you handed him the whole direction, in public, three weeks into the job.
>
> This is the most under-discussed failure mode for new leads who are *afraid of seeming arrogant*: you over-correct so hard that you teach the room your authority is available to anyone who pushes on it firmly enough. **Accommodate** quadrant — you sacrificed your own position to dissolve the discomfort of having been challenged. The cost isn't this conversation. It's that the next person who disagrees with you now knows the move is to disagree *loudly*, because loud disagreement gets rewarded with control.
>
> **AID**: your *action* was over-conceding to absorb the tension; the *impact* is a team that's now unsure who's actually deciding and a senior IC who feels handed an authority he never wanted; the *do-differently* is to credit the specific point without surrendering the role. "You're right about screen 2 — let's fix that one, the rest holds" gives Marcus the full win he actually asked for, and keeps you the lead. The credit was free. The abdication was expensive. You only needed the first.

---

### Decision 2 — Marcus, One-on-One

D1 was the **public** moment — the whole team watching how the new lead handles being challenged. D2 is **private**: a 1-on-1 DM thread with Marcus the next morning. The crit is settled enough; now it's about the *relationship* underneath it. Eight years your senior, just made your report, and how you handled the room on Tuesday determines exactly which Marcus shows up in this DM.

**The decision is shared.** Only the **first incoming line swaps** based on `choiceHistory[0]` — Marcus arrives in a different posture depending on how D1 went. Everything after that first line, and all five response options, are the same across paths. (See *Notes for the Code Agent* for the `incomingVariants` shape.)

**Incoming messages (stream in before player can respond):**

The first message is a **framing variant** — pick by `choiceHistory[0]`:

```
VARIANT [if D1 = fighter] — entrenched / formal (you ruled him out of order in public)
1a. Marcus · 9:40 AM (typingDelay: 3000ms)
    "Morning. Sent the second-screen notes to your inbox — kept it to the thread like you asked. Figured I'd give you a heads up here so it's not a surprise. I'll keep feedback in writing from here on out; cleaner that way. Let me know if you need anything else from me on it."

VARIANT [if D1 = negotiator] — competitive / proving-it (you made it a design-off)
1b. Marcus · 9:40 AM (typingDelay: 3000ms)
    "Morning. Counter for screen 2 is basically done — I'll have the full thing to you by Thursday like we said. I built it side-by-side against the original so the comparison's fair. Quick question before I polish it: when you said 'if it tests better it wins' — who's the judge if there's no test? Just want to know the rules of the game I'm playing."

VARIANT [if D1 = diplomat] — warmed / collaborative (you asked him to show it)
1c. Marcus · 9:40 AM (typingDelay: 2500ms)
    "Morning. Yesterday was good — Tara said she finally gets why the pattern fails on mobile, which honestly made my month. I owe you something though, and I'd rather say it in DM than let it sit. Got a sec?"

VARIANT [if D1 = avoider] — patient-but-stalled (you deferred and it drifted)
1d. Marcus · 9:40 AM (typingDelay: 3000ms)
    "Morning. Not nagging, promise — but I still don't have a direction on screen 2 and the milestone's Monday. I can build either version, I just need you to pick one. I think there might also be a thing worth saying about Tuesday that's separate from the screen. Do you have a few minutes for an actual conversation?"

VARIANT [if D1 = people-pleaser] — uncomfortable / over-given-to (you handed him the wheel)
1e. Marcus · 9:40 AM (typingDelay: 3000ms)
    "Morning. So — I've been sitting with yesterday and I need to say something straight, because I don't think you meant for it to land the way it did. You handed me the whole direction in front of the team after I flagged one screen, and that put me in a weird spot I didn't ask to be in. Can we talk about it for real?"
```

Then the **shared** continuation (same for every path):

```
2. Marcus · [+1 min from variant] (typingDelay: 2500ms)
   "Here's the thing I actually want to say, lead to... well, you're my lead now, which is still new for both of us. I've got eight years on you in this. That doesn't make me right — it makes me *experienced*, and those aren't the same thing, I know that better than anyone. I'm not trying to test you. But I'd be lying if I said it isn't a strange adjustment. So: how do you want this to work between us?"
```

**Response Options:**

> **Note for the Code Agent:** D2 uses the `incomingVariants` field exactly as Scenario 1's D2 does — keyed to `choiceHistory[0]`, the variant renders **first**, then the shared message #2 streams. Timestamps: all variants are stamped 9:40 AM Wednesday for simplicity (the public D1 has converged onto a next-morning 1-on-1 regardless of which D1 path was taken). Shared #2 = "[+1 min from variant]" → stamp 9:41 AM.

---

#### 🔥 Fighter

**Button text:**
> "Honestly? Direct. The eight years are real and I'm not going to pretend I have your reps — but I'm the lead, and the way it works is I make the calls and own them. I want your dissent. In crit, in DM, loudly if you want. But once I decide, I need you building it, not relitigating it. Can you do that?"

**Consequence chat items:**

```
1. (pause: no typing indicator for 3500ms)
2. Marcus · 9:46 AM (typingDelay: 0ms)
   "I can do that. To be clear, I've never not built the decided thing. That was never the question."
3. Marcus · 9:47 AM (typingDelay: 2500ms)
   "The question was whether you wanted the dissent or just wanted me to know you outrank me. I think you just answered it. Building the decided thing. 👍"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "asked how we should work together and got 'i'm the lead, i make the calls.' which — yeah, obviously, i've never disputed that, i've shipped under six leads. i was offering to actually align and got reminded of the org chart i already know by heart. fine. i'll build what's decided, keep my mouth shut, and let them find out about the screen-2 stuff the expensive way. eight years and i get the 'can you follow direction' talk. cool."
```

**Future You:**

> You set a boundary, and a new lead *does* sometimes have to say "I decide and I own it" out loud — especially to a report with more tenure, because the ambiguity can otherwise calcify into a permanent two-captains problem. There's spine here, and spine is worth real money.
>
> But read what Marcus actually offered and what you answered. He asked "how do you want this to work *between us*?" — an invitation to build the relationship — and you answered with a chain-of-command refresher he didn't need. He told you, twice, that he's never refused to build a decided thing. So the boundary you defended wasn't actually under threat; you asserted authority against a challenge that wasn't there, which is the exact tell of someone who's *worried* about their authority rather than secure in it. **Thomas-Kilmann Compete** against a phantom. On the **Radical Candor** grid you challenged hard and skipped care, and the cost is precise: he just decided to stop volunteering the expensive things he knows. Your most experienced person is going to start *withholding* the eight years instead of lending them to you.
>
> Future you wants to ask: you proved you outrank him. Did you ever doubt it? And if you didn't — who was that speech actually for?

---

#### ⚖️ Negotiator

**Button text:**
> "Here's what I'd propose: you're my most senior person, so let's make that official — I want you as my de facto design advisor on the big calls. You get a real voice early, before things hit crit. In exchange, when I make the final call, you back it in the room even if it's not your pick. You get influence, I get a united front. Deal?"

**Consequence chat items:**

```
1. Marcus · 9:45 AM (typingDelay: 2500ms)
   "...that's actually a good offer. Early voice is what I wanted. I can back a call I disagreed with if I was genuinely in the room for it."
2. Marcus · 9:46 AM (typingDelay: 3000ms)
   "One honest note, and then I'll take the deal: we just turned 'how do we respect each other' into a trade. I asked how this works between us and the answer was a transaction — influence for loyalty. It's a fair transaction. It just tells me the relationship runs on terms, not trust, yet. Which, fine, three weeks in. We'll get there. Deal."
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "got made 'design advisor' which is genuinely good and i'm not going to pretend it isn't — early voice is the whole thing i was missing. but it came as a deal. influence in exchange for backing them in the room. i traded for a thing that, honestly, a senior person on the team should just... have? felt like negotiating a salary for a job i thought i already had. i'll take it, it's a good deal. just clocked that we did a contract instead of an actual conversation about working together. one step at a time i guess."
```

**Future You:**

> Genuinely shrewd. You took a fuzzy relational question and converted it into a structure with upside for both of you — Marcus gets the early voice he was clearly starving for, you get a unified front in the room, and you did it without either folding or fighting. That's the **Compromise** quadrant, and as a *mechanism* it's good: an advisor role for your most senior IC is a smart org move.
>
> But Marcus isn't a peer or a vendor — he's your report, and you just taught him that influence, a voice, and respect are things he *bargains for* rather than things you extend because he's earned them. He said it cleanly: he asked how the relationship works and you handed him terms. **SBI**: the *situation* was an invitation to build trust; your *behavior* was to offer a transaction; the *impact* is he got real upside but learned the relationship "runs on terms, not trust, yet." A trade resolves the *structure*. It doesn't build the *thing he was actually asking for*.
>
> The advisor role was the right idea. Next time, *give* it — "I want you in early on the big calls, full stop" — and ask for the united front as a value you both hold, not a price he pays. The deliverable was great. You just skipped the free part: telling a senior person you respect them without making it contingent.

---

#### 🤝 Diplomat

**Button text:**
> "Straight answer, because you gave me one: the eight years scare me a little, and I'd rather admit that than overcompensate by pretending they don't matter. Here's what I want — your experience pointed *at the work*, not at the org chart, and not swallowed because I'm nominally above you. I'll make the final calls and own them. But I want to be the lead who has the most experienced person in the room actually *fighting for the work* in front of me. Tuesday, minus the surprise, is exactly what I want more of. Does that work for you?"

**Consequence chat items:**

```
1. Marcus · 9:44 AM (typingDelay: 3000ms)
   "...okay. That's the most honest thing a lead has said to me in a while."
2. Marcus · 9:45 AM (typingDelay: 2500ms)
   "'Tuesday minus the surprise' — yeah. I can do the substance without the ambush. I'll bring you the hard stuff early and in private when it's a bomb, and loud in crit when it's just a good argument. And I'll tell you when I think you're wrong, which means you can actually trust it when I tell you you're right."
3. Marcus · 9:46 AM (typingDelay: 2500ms)
   "For what it's worth — you saying the eight years 'scare you a little' is exactly why they shouldn't. People who can name that are the ones who end up actually leading the senior folks instead of managing around them. I'm in."
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "ok i need to recalibrate on this one. i went in ready to manage a defensive new lead and instead got 'the eight years scare me a little and i'd rather say that than overcompensate.' do you know how rare that is? every insecure lead i've had buried that and let it leak out as control. this one just said it out loud and then asked me to keep fighting for the work. i'd give this person my actual best thinking, not the sanded-down version i give people i'm managing carefully. three weeks in. genuinely impressed. don't tell them i said that."
```

**Future You:**

> This is the repair, and it's the rarest one to pull off because it required you to do the thing every new lead is most afraid of: name your own insecurity *to the person who triggered it*, out loud, without it becoming either a confession that undermines you or a performance that manipulates him. You said "the eight years scare me a little" and then immediately pointed past it — to the work, to what you actually want from him. That's not weakness. That's the **Collaborate** quadrant and the full **Radical Candor** square at the same time: you cared personally (named the real thing) *and* challenged directly (I make the calls, you fight for the work).
>
> Watch what it bought you, in his own words to Lena: he just decided to give you his *actual best thinking* instead of the sanded-down version people give a lead they're carefully managing. That is the entire reason you'd ever want a report with eight more years than you — and most new leads forfeit it in week one by treating that seniority as a threat to neutralize instead of a resource to unlock.
>
> Two things to carry. One: this only works because you *meant* it — "the eight years scare me a little," said as technique, curdles into manipulation faster than anything in the kit, and a designer with Marcus's reps can smell it. Two: naming the dynamic doesn't dissolve it. You'll still have to *make a call he hates* someday, and "fight for the work" has to survive that day too. Repair built the trust. Now you have to spend it correctly. But you spent ten minutes and a little ego, and you got the best designer on your team genuinely on your side. Cheapest authority you'll ever buy.

---

#### 🚪 Avoider

**Button text:**
> "Honestly this is a great conversation to have, and I want to give it the time it deserves rather than knock out a quick DM answer between meetings. Can we put thirty minutes on the calendar later this week and actually map out how we want to work together? I don't want to wing something this important."

**Consequence chat items:**

```
1. Marcus · 9:44 AM (typingDelay: 2500ms)
   "Sure. Later this week works."
2. Marcus · 9:45 AM (typingDelay: 2500ms)
   "I'll send an invite so it actually lands on a calendar and doesn't become a someday thing. No offense — I've had a lot of 'let's find time to align' that quietly never found time."
3. TimeMarker: "— Friday, 8:55 AM —"
4. Marcus · 8:55 AM (typingDelay: 2000ms)
   "Still good for our 30 today? 🙂 (Also — screen 2 milestone is Monday, so it'd be great to actually land the working-relationship thing before the next crit, not after.)"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "asked how we should work together and got 'let's put 30 min on the calendar to do it properly,' which could be respect or could be a polite slow-no. genuinely can't tell yet. i sent the invite myself so it can't evaporate. the optimist read is 'they take this seriously enough to schedule it.' the cynic read is 'they don't want to have it.' i'm choosing optimist for now but the milestone's monday and i'd like to know who i'm working for before then, you know?"
```

**Future You:**

> Wanting to give a relationship-defining conversation real time instead of a between-meetings DM is genuinely good judgment — these talks *do* go better with thirty unhurried minutes than thirty rushed seconds, and refusing to wing something important is the **Avoid** quadrant used legitimately.
>
> But notice the two things Marcus did in response: he had to *send the invite himself*, and he attached a deadline you didn't ("before the next crit, not after"). To a senior person extending an honest "how do you want this to work?", a deferral is a *coin flip* — half of him hears "they respect this enough to schedule it," the other half hears "polite slow-no." You bought yourself a better conversation and simultaneously bought yourself a Friday-morning tripwire: that 8:55 AM "still good?" ping is the sound of someone protecting himself against the version where you don't show. The defer is only honest if the follow-through is non-negotiable, and you've now made Friday's 30 minutes load-bearing for whether your most experienced report decides you're deliberate or evasive.
>
> Show up sharp on Friday, or this becomes the "let's find time" that proves his cynic read right. You didn't avoid the conversation. You scheduled the test.

---

#### 🙏 People Pleaser

**Button text:**
> "Oh gosh, please don't feel weird about any of it — you should ABSOLUTELY keep speaking up, I'd never want you to hold back because of some title thing. Honestly you probably should've gotten this role over me, you've earned it way more. Let's just keep things exactly how they've always been, I don't want the lead thing to change anything between us 🙏"

**Consequence chat items:**

```
1. Marcus · 9:44 AM (typingDelay: 3000ms)
   "Whoa — okay, no. I don't think you should've gotten my role, and I definitely don't think you should've not gotten yours. You're the lead. That part's real and I need it to be real."
2. Marcus · 9:45 AM (typingDelay: 3000ms)
   "'Keep things exactly how they've always been' — that's the thing I can't do. You're not my peer anymore, you're my lead, and if you can't hold that, then every call you make has a question mark on it, including the ones I agree with. I asked how this works and you kind of said 'let's pretend it didn't change.' I need you to be in charge so I can stop wondering who is."
3. TimeMarker: "— 9:52 AM —"
4. Tara · 9:52 AM (typingDelay: 2500ms)
   "hey sorry to interrupt — is screen 2 a 'marcus decides' thing now or a 'you decide' thing? getting slightly different signals and just want to build the right one 😅"
```

**DM intercepted:**

```
From: Marcus
To: Lena
Text: "this one's eating at me. i asked how we work together now and got 'you should've gotten the role, let's keep everything the same.' which is meant kindly and is somehow the most destabilizing thing they could've said?? i don't want them insecure-and-controlling but i ALSO need them to actually BE the lead. 'let's pretend nothing changed' isn't humility, it's a vacuum, and vacuums on a design team get filled by whoever's most confident in the room — which right now is me, by default, which is bad for everyone. i need a boss, not a fan. somebody has to drive."
```

**Future You:**

> Your heart is in the most generous place in the building — you want Marcus to feel valued, unthreatened, and free to speak, and that *care* is the raw material of every leader worth following. So hear the next part as aim, not indictment: you over-gave so hard you gave away the one thing he was actually asking you to keep.
>
> He asked "how does this work between us?" and you answered, essentially, "let's pretend it didn't change" — plus "you should've gotten my role," which you meant as humility and which landed as a vacuum. Watch his response: *he pushed the authority back at you.* "I need you to be in charge so I can stop wondering who is." That's the tell people miss — a good senior IC doesn't actually want a leaderless team; ambiguity about who decides makes *his* job harder, puts a question mark on every call, and (per Tara's interruption) leaves the team building toward two different signals. **Accommodate** quadrant: you sacrificed your own position to dissolve the discomfort of authority, and the discomfort didn't go away — it spread to the whole team as "wait, who's deciding?"
>
> **AID**: the *action* was disclaiming your own role to make Marcus comfortable; the *impact* is a destabilized senior IC, a confused mid-level, and a leadership vacuum that defaults to whoever's most confident in the room; the *do-differently* is to separate *valuing him* from *abdicating to him*. "You're the most experienced person here and I want your voice loud — *and* I'm the lead, and I'll own the calls" gives him 100% of the respect and 0% of the vacuum. The warmth was free and true. The abdication was expensive and false. He only wanted the first one — he told you so.

---

### Decision 3 — The Next Crit, And Sam Is In The Room

It comes back around. **One week later**, the next design critique — same channel reacts after, but this time the stakes compound: the screen-2 work is on the line *for real* (the milestone shipped Monday with a version of it), **Sam (your manager) has quietly joined the crit** to see how the new lead is settling in, and Marcus has new feedback. How the room reads this moment depends entirely on what you built with him over the last week.

**Incoming messages (stream in before player can respond):**

```
1. TimeMarker: "— Tuesday, 3:02 PM · #crit-aftermath —"
2. Tara · 3:02 PM (typingDelay: 2000ms)
   "round 2 of the famous crit channel 🍿 (kidding. mostly.)"
3. Sam · 3:03 PM (typingDelay: 2500ms)
   "hey all — i sat in on crit today, just getting a feel for how the team's running under new leadership. don't mind me. 🙂 (it's going well, fwiw.)"
4. Marcus · 3:05 PM (typingDelay: 3000ms)
   "So — screen 2. We shipped a version Monday. I've now got real session data, and there's a thing in it I think we got wrong, including possibly the part *I* pushed for. I want to raise it, but it's your room. Do you want me to lay it out here, or take it offline?"
5. Tara · 3:06 PM (typingDelay: 1500ms)
   "👀"
6. Sam · 3:06 PM (typingDelay: 1500ms)
   "👀 (genuinely curious how you'll run this)"
```

> **Note for code agent:** D3's `incoming` is **fully shared** (no variants). Sam appears in-channel here as the manager observing — render as **"Sam"** (existing `SM`, `#5F6368`). The branching lives **per option**, via lock conditions and compounding modifiers keyed to `choiceHistory`, exactly like Scenario 1's D3. See the consolidated table in *Notes for the Code Agent* below. The room is watching, and this time so is your boss. This is where D1 and D2 compound. Note the twist baked into Marcus's #4: the flaw in the data includes *his own* prior pick — so this isn't "Marcus vs. you" again; it's a test of whether you built a team that can be wrong *together* in public.

**Response Options:**

---

#### 🔥 Fighter

**Button text:**
> "Lay it out here, Marcus. If we got it wrong — including the part you pushed — I'd rather the team watch us catch it in the open than quietly patch it later. Bad calls don't get to hide just because they're inconvenient. Go."

**LOCK CONDITION:** *Never locked.* (You can always choose to call it out in the open — the only question is whether the room reads it as confident leadership or as the same combativeness from before, which depends on prior choices.)

**Consequence chat items:**

```
BASE:
1. Marcus · 3:08 PM (typingDelay: 2500ms)
   "Okay. Here it is — the nav fix helped desktop but the session data shows mobile users still dead-ending on screen 2, including with my version. So the original instinct AND my counter both missed the actual problem. I think it's the entry point, not the layout."

CONDITIONAL — append based on history:

[if choiceHistory[1] = diplomat]  (you built real trust with Marcus in D2)
2. Marcus · 3:09 PM (typingDelay: 2500ms)
   "And I want to say in front of everyone — I'm only comfortable putting my own miss on the table because of how the last two weeks went. This is what 'fight for the work, not the ego' looks like from my side. Here's the data."
3. Sam · 3:10 PM (typingDelay: 2500ms)
   "...okay, THAT is the healthiest crit I've seen on this team in two years. Your most senior person just publicly debugged his own call. Whatever you've been doing — that's the job. 👏"

[if choiceHistory[1] = fighter OR choiceHistory[1] = people-pleaser]  (you asserted rank, or abdicated, in D2)
2. (pause: no typing indicator for 4000ms)
3. Marcus · 3:10 PM (typingDelay: 0ms)
   "Data's in the doc. I'll keep it factual."  ← he lays out the problem, correctly, with zero warmth and no air-cover for you
4. Sam · 3:11 PM (typingDelay: 2500ms)
   "good that it's surfacing. the temperature in here is a little... formal, though. i'll catch you after, want to understand the dynamic."

[if choiceHistory[0] = fighter]  (you also ruled him out of order in D1)
5. Sam · 3:12 PM (typingDelay: 2500ms)
   "one thing i'll name gently — i've now seen two crits where the energy with marcus runs hot. i trust your calls. i want to understand why it keeps being a temperature thing. let's talk after."
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Sam
To: (you)
Text: "that was genuinely impressive. you created a room where your most experienced person felt safe enough to say 'i was wrong' out loud, in front of his manager AND yours. that doesn't happen by accident — that's three weeks of you not making it about who's in charge. keep doing exactly that."

[otherwise]
From: Marcus
To: Lena
Text: "called the screen-2 miss out in the open like they asked, including my own part in it, because it's the right thing for the work and i'm a professional. did i go one degree warmer than the data required, with sam sitting right there? i did not. 'lay it out here' is easy to say when you haven't built the kind of room where someone wants to. the data's correct. that's all they're getting from me today."
```

**Future You:**

> Calling the miss into the open — *including Marcus's own piece of it* — was the right instinct and a brave one, especially with Sam watching. You modeled that bad calls don't get to hide, which is exactly the culture a design team needs. **Compete** quadrant, but pointed at the *right* target this time: the flaw, not the person.
>
> But a public reckoning is only as good as the room you've built to hold it. If you spent the last two weeks building real trust with Marcus, he doesn't just present the data — he *publicly debugs his own call* and credits the environment you made, and Sam watches your most senior IC do the single healthiest thing a senior IC can do. That's the whole game. If instead you asserted rank or abdicated in D2, Marcus will still surface the problem — correctly, professionally — but with zero warmth and no air cover, and Sam will feel the chill and pull you aside to "understand the dynamic." And if you also ran hot in D1, that's now a *pattern* Sam has seen twice, which is a very different conversation than a one-off.
>
> Conviction in the room is only credible when the room is behind you. "Lay it out here" is a great line — but whether it sounds like leadership or like the third round of a fight was decided two weeks ago. Friday's standing was set on the prior Wednesday.

---

#### ⚖️ Negotiator

**Button text:**
> "Let's do this efficiently: lay out the headline here so the team has context, then you and I take the deep dive offline and bring a single recommendation back tomorrow. The team builds on a decision, not a debate. Fair split?"

**LOCK CONDITION:** *Never locked,* but the framing shifts (see conditional). If D2 already turned the relationship transactional (`choiceHistory[1]` = negotiator), Marcus reads this as another deal and names the pattern.

**Consequence chat items:**

```
BASE:
1. Marcus · 3:08 PM (typingDelay: 2500ms)
   "Works for me. Headline: mobile users still dead-end on screen 2, including with my version — it's the entry point, not the layout. Full breakdown offline, recommendation tomorrow."
2. Sam · 3:09 PM (typingDelay: 2000ms)
   "clean. 'context now, decision tomorrow' is a good way to run a room. 👍"

CONDITIONAL:

[if choiceHistory[1] = diplomat]  (you built trust in D2 — the deal lands as alignment, not transaction)
3. Marcus · 3:10 PM (typingDelay: 2500ms)
   "And I'll say the quiet part: I'm bringing you my own miss on this, not defending my counter. That's the version of 'advisor' that's actually worth something. Recommendation tomorrow, jointly."
4. Sam · 3:11 PM (typingDelay: 2000ms)
   "love that. 🙂"

[if choiceHistory[1] = negotiator]  (you also made D2 a trade — Marcus clocks the pattern)
3. Marcus · 3:10 PM (typingDelay: 2500ms)
   "Noticing we keep solving things with a clean split — context here, deep dive there, deal, deal, deal. It's efficient, I'll give you that. One day I'd like to just... think a problem through with you, no terms. But yeah. Tomorrow."

[if choiceHistory[0] = avoider]  (D1 was a stall — the team is wary of 'offline')
5. Tara · 3:12 PM (typingDelay: 2500ms)
   "offline + recommendation tomorrow works as long as it's a real tomorrow — last screen-2 call took a few days to land and the milestone snuck up 😅 just flagging"
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Marcus
To: Lena
Text: "they ran it as 'headline now, decision tomorrow' and it actually felt like a partnership, not a punt. i offered up my own miss without it being a thing. funny how the same 'let's split this' move feels totally different now that there's trust under it. two weeks ago it'd have felt like being managed. today it just felt like working together. growth, i guess. theirs and maybe mine."

[otherwise]
From: Marcus
To: Lena
Text: "another clean split — context here, deep dive offline, recommendation tomorrow. it's a good process, genuinely. i just keep noticing that every interaction with this lead resolves into a transaction. efficient leadership, sure. i'd trade a little efficiency for the sense that we're actually thinking together instead of trading turns. but the work's getting done and sam looked happy, so. can't argue the results."
```

**Future You:**

> Skilled, and well-timed with Sam in the room — you gave the team context without letting crit spiral into a live debate, and you converted a charged moment into a clean process: headline now, decision tomorrow. That's the **Compromise** quadrant running exactly as designed, and Sam clocked it as a good way to run a room.
>
> The risk is the same one negotiation always carries with a *report*: if the relationship is already built on trust, "let's split this" lands as partnership — Marcus even offers his own miss freely, and the deal is just the shape of how you collaborate. But if you made D2 a transaction too, Marcus is now noticing the *pattern* — every interaction resolves into terms — and he says it out loud: he'd trade a little efficiency to *think a problem through with you, no deal*. **SBI**: the *situation* keeps being a chance to build a thinking partnership; your *behavior* keeps being to structure it as an exchange; the *impact* is a senior IC who's productive but quietly wishes the relationship ran on trust instead of terms.
>
> A split is the right move when the thing genuinely has two sides to divide. Just make sure you're not *defaulting* to a deal because it's the move you're most comfortable with — sometimes the highest-value thing you can do with your most experienced designer is just think out loud together with nothing on the table.

---

#### 🤝 Diplomat

**Button text:**
> "Lay it out here — and Marcus, thank you for flagging your own piece of it, that's exactly the room I want this to be. Team, this is the standard: we follow the data even when it costs us the thing we argued for. Walk us through it, and let's decide together what ships next."

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[1]` = fighter OR people-pleaser** *(you either asserted rank over Marcus or abdicated to him in D2 — in both cases you never built the trust that makes "the room I want this to be" a true sentence)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't say 'exactly the room I want this to be' — you haven't built that room. After D2, Marcus either learned to keep his head down (you pulled rank) or doesn't know who's actually deciding (you abdicated). The 'we follow the data together' story requires trust you didn't bank. (It was available — the door was open in that 1-on-1.)"*

**Consequence chat items:**

```
BASE:
1. Marcus · 3:08 PM (typingDelay: 2500ms)
   "Appreciated. Here it is — mobile still dead-ends on screen 2, including with my version, so my counter missed the real issue too. It's the entry point. I'd rather own that publicly than have us quietly find it in the metrics next quarter."
2. Sam · 3:09 PM (typingDelay: 3000ms)
   "...I want to mark this moment. Your most senior designer just said 'I was wrong' in front of his manager, voluntarily, because the room is safe enough for it. That is the single hardest thing to build on a team and you built it in three weeks. 👏"

CONDITIONAL:

[if choiceHistory[0] = diplomat]  (you handled D1 publicly with grace too — the whole arc is coherent)
3. Tara · 3:11 PM (typingDelay: 2500ms)
   "ok as the person who held her breath during The Original Crit — this is such a different room than two weeks ago. i'm actually learning a ton watching you two go at the work like this 🙂"
4. Marcus · 3:12 PM (typingDelay: 2000ms)
   "Entry-point fix in 30. Tara, want to pair on it? Good one to learn on."

[if choiceHistory[0] = fighter]  (you ran hot in D1 but repaired in D2 — recovery arc)
3. Sam · 3:11 PM (typingDelay: 2500ms)
   "and honestly — i saw a rougher version of this room two weeks ago. whatever you adjusted with marcus since then, it took. that's a leader who course-corrects, which i'll take over one who never needed to."
```

**DM intercepted:**

```
[if choiceHistory[1] = diplomat]
From: Marcus
To: Lena
Text: "i just admitted i was wrong about my own counter, out loud, with TWO levels of management watching, and it felt completely safe. do you understand how rare that is? i've spent eight years on teams where being wrong in public was a career risk. this lead built a room where it's just... how we get to the right answer. three weeks. i'd run through a wall for this person and i don't say that lightly. the eight years thing? i don't even think about it anymore. we're just doing the work."

[otherwise]
From: Marcus
To: Lena
Text: "good crit. surfaced the real problem, including my own miss, and the lead handled it well in the room. i'll be honest though — there's a warmth in how they ran it today that wasn't fully there in our 1-on-1, and i'm still calibrating whether it's real or whether it's for sam's benefit. probably real. i'm a little guarded still. that's on the history, not today."
```

**Future You:**

> This is the ceiling of the entire scenario. You took the highest-stakes possible moment — your most senior report flagging a flaw *that includes his own prior pick*, with your manager watching — and you made it a demonstration of the culture instead of a referendum on your authority. You thanked him for owning his piece, you named the standard for the whole team, and you handed the room a shared problem instead of a power contest. **Collaborate** quadrant, full **Radical Candor** square, in front of your boss. And Sam said the quiet part out loud: a team where the most senior person will say "I was wrong" voluntarily is the hardest thing in the world to build, and you built it in three weeks.
>
> But notice *why* this door was even open to you. It was locked the moment your D2 went to rank-pulling or abdication — because "exactly the room I want this to be" is only a true sentence if you actually built the room, and you build it in the unglamorous 1-on-1, not in the crit with an audience. If you repaired honestly in D2, Marcus *volunteers* his own miss and the whole arc clicks into place; if you didn't, you can't get here at all, no matter how good the words sound. The best move in the highest-stakes room was set up two weeks earlier, in private, with nobody watching.
>
> Two cautions, because you earned the right to hear them. One: this only works because you meant it — performed safety is more corrosive than honest authority, and Marcus would clock the difference instantly. Two: the room you've built now has to survive the day you make a call Marcus genuinely hates and *don't* fold. Safety isn't agreement. Keep deciding. You just made it so that when you do, the most experienced person in the building is arguing *for* the work, not *against* you.

---

#### 🚪 Avoider

**Button text:**
> "Let's not unpack live data in crit on the fly — take it offline, dig into it properly, and bring me the findings when they're solid. I don't want to react to a headline number in the room. Loop me in once you've got the full picture."

**LOCK CONDITION:** *Never locked,* but the room's read depends on history (see conditional). After a pattern of deferring, the same move reads as a flinch rather than rigor — especially with Sam in the room.

**Consequence chat items:**

```
BASE:
1. Marcus · 3:08 PM (typingDelay: 2500ms)
   "Sure. I'll write it up properly and bring it to you. Just flagging it's time-sensitive — mobile users are dead-ending right now, in prod."
2. Sam · 3:09 PM (typingDelay: 2500ms)
   "offline's reasonable for the deep numbers — just make sure 'when it's solid' has a date on it, since it's live in prod. 🙂"

CONDITIONAL:

[if choiceHistory[0] = avoider]  (you also deferred the original screen-2 call in D1)
3. Tara · 3:10 PM (typingDelay: 2500ms)
   "gentle flag — the FIRST screen-2 decision also went 'offline, decide later' and the milestone kind of snuck up on us. could we put an actual time on this one? 😅"
4. Sam · 3:11 PM (typingDelay: 2500ms)
   "i'll sit in on the offline so it lands — this is the second 'take it offline' on screen 2 and i'd like a decision to actually come out the other side this time."

[if choiceHistory[1] = avoider]  (you also deferred the D2 relationship talk)
5. Marcus · 3:11 PM (typingDelay: 2500ms)
   "I'll have it to you by tomorrow AM with a recommendation, so it can't drift. (Trying to make the offline a real one.)"  ← he's quietly building the structure you keep deferring

[if choiceHistory[1] = diplomat]  (you built trust in D2 — he shortcuts it for you)
3. Marcus · 3:10 PM (typingDelay: 2000ms)
   "I'll have the write-up to you in an hour — it's mostly done, I just didn't want to dump raw data in the room. You'll have a real recommendation, not a headline."
```

**DM intercepted:**

```
[if choiceHistory[0] = avoider OR choiceHistory[1] = avoider]
From: Sam
To: (you)
Text: "real talk, between us — this is the second time screen 2 has gone 'let's take it offline.' once is good judgment. a pattern starts to look like the new lead doesn't want to decide in the room, and crit is exactly where a lead is supposed to be able to. i'm not worried yet. but make this offline produce a real decision, with a date, so i don't have to wonder. you've got this — just close the loop."

[otherwise]
From: Marcus
To: Lena
Text: "lead punted the live-data thing to offline, which honestly? fair — you shouldn't debug session metrics on the fly in front of everyone. i just hope 'offline' is a real plan and not a fog. it's live in prod and people are dead-ending right now. i'm writing it up with a recommendation either way so there's something concrete to decide on. someone has to make the offline real."
```

**Future You:**

> Refusing to debug live session data on the fly, in front of the whole team, is a genuinely defensible instinct — a hot take on a headline number is worse than a careful read, and the **Avoid** quadrant is correct when the cost of a sloppy in-room reaction is high and the real analysis is a write-up away.
>
> But avoidance has a counter, and with Sam in the room you may have just triggered it. A single "take it offline" is rigor; a *pattern* of deferring screen-2 decisions is a reputation, and Sam names it precisely — crit is exactly where a lead is supposed to be able to decide, and a new lead who keeps moving the decision elsewhere starts to look like someone who doesn't want to make it. Worse, notice who keeps filling the vacuum: if you deferred in D1 and D2 too, *Marcus* is the one quietly attaching deadlines and writing the recommendation — doing the deciding you keep declining, with less authority and more frustration than either of you needs. And it's *live in prod*, so the cost of the deferral isn't abstract; users are dead-ending while the decision waits.
>
> A defer is a loan against your credibility, and you can carry one. But by the second deferral on the same problem — with your boss watching — Sam starts wondering whether you're *careful* or whether you *can't decide in the room*, and that's a question you never want asked about you instead of to you. If you defer, put a clock on it and make the offline produce a real, dated decision. An unspecific "loop me in when it's solid" is how rigor turns into a flinch.

---

#### 🙏 People Pleaser

**Button text:**
> "You know what, Marcus, you clearly know this better than I do — why don't you just take screen 2 and run with whatever you think is right? I trust you completely on this one. The team's in great hands with you driving it 🙌"

**LOCK CONDITION:** 🔒 **Locked if `choiceHistory[1]` = diplomat OR fighter** *(in D2 you either built a real working relationship OR explicitly claimed the lead role — in both cases handing Marcus the wheel now would visibly contradict the relationship you just established, with Sam watching the contradiction)*.
> Locked-state copy (greyed out, not clickable):
> *"🔒 You can't hand Marcus the wheel now — you spent D2 establishing that you're the lead who makes the calls (or genuinely partnering as equals). Abdicating in front of Sam would contradict your own week-ago self out loud. (This reflex is only on the board when you never planted the flag.)"*

> **Design note:** This is Scenario 3's central Bandersnatch lock. The "hand it to the more experienced person" reflex is *removed from the board* the moment you've already claimed or co-built the authority — because abdicating then is a visible, witnessed contradiction. If D2 was Negotiator (relationship runs on terms, role never fully claimed), Avoider (you deferred the role conversation entirely), or People-Pleaser (you already abdicated once), the option is available — but it's still a costly choice, not a free one (see below).

**Consequence chat items (only reachable when NOT locked — i.e. `choiceHistory[1]` ∈ {negotiator, avoider, people-pleaser}):**

```
BASE:
1. Marcus · 3:08 PM (typingDelay: 3000ms)
   "...I appreciate the trust, but — in front of the team and with Sam here, I don't think 'Marcus, you run it' is the read you want. It's your call to make. I'm happy to recommend. I shouldn't be the one deciding what the new lead's team ships."
2. Tara · 3:10 PM (typingDelay: 2500ms)
   "yeah honestly i'm a little confused on who's steering screen 2 at this point 😶 not trying to make it weird, just want to build the right thing"

CONDITIONAL:

[if choiceHistory[1] = people-pleaser]  (you ALSO abdicated in D2 — this is now a clear pattern, in front of Sam)
3. Sam · 3:12 PM (typingDelay: 3000ms)
   "i'm going to gently step in — marcus, hold for one sec. i want to make sure the team knows who's making the screen-2 call, because right now it's reading as nobody, and that's the one thing a crit can't end on. let's you and i talk right after, yeah?"  ← Sam is quietly stepping in to fill the authority vacuum you left

[if choiceHistory[1] = negotiator]  (you traded in D2 — Marcus invokes the actual deal)
3. Marcus · 3:11 PM (typingDelay: 2500ms)
   "Also — the deal was *early voice*, not *the wheel*. I back your calls in the room; that's my half. But there has to be a call from you to back. Give me a direction and I'll defend it."  ← he holds you to the structure you built

[if choiceHistory[0] = diplomat OR choiceHistory[0] = fighter]  (you DID plant a flag in D1, then abdicate now — visible whiplash)
4. Sam · 3:13 PM (typingDelay: 2500ms)
   "for what it's worth — two weeks ago you held a direction firmly in this exact channel. today you're handing it off entirely. i'm not saying either's wrong, but the team's getting two different signals about how you lead. worth getting consistent. let's chat."
```

**DM intercepted:**

```
[if choiceHistory[1] = negotiator]
From: Marcus
To: Lena
Text: "lead tried to hand me the entire screen-2 call in front of sam and i had to publicly decline?? the whole point of the advisor deal was i get EARLY VOICE, not that i secretly run the team while they get the title. i don't want to be the power behind the throne, i want there to be a throne with someone on it. had to remind them of their own deal. with their boss watching. brutal. they're a good designer who keeps trying to give away the part of the job that's actually theirs."

[otherwise]
From: Sam
To: (you)
Text: "hey — no alarm, but i want to flag what i saw. handing the screen-2 call to marcus in the room, with me sitting there, read as the new lead not wanting to own a decision. marcus is great, but if your most senior IC is making the calls, the team stops knowing what your role is — and so do i. you don't have to decide everything. but you have to be visibly the one deciding *something*. let's grab 20 min."
```

**Future You:**

> When this option is even *available* to you, the instinct underneath it is real humility — Marcus genuinely does have more reps, and trusting your expert is a virtue, not a vice. Hold onto that. The problem isn't the respect. It's that you converted "I trust your read" into "you run it," in public, with your manager in the room — and that's not delegation, it's a leadership vacuum with a bow on it.
>
> Watch how everyone responds, because they're all telling you the same thing from different angles. *Marcus declines* — he doesn't want to be the unelected captain; he even points out it's "not the read you want" with Sam there. *Tara gets confused* about who's steering. And *Sam* (in the DM) names it cleanly: you don't have to decide everything, but you have to be *visibly the one deciding something*, or the team stops knowing what your role is. **Accommodate** quadrant: you tried to dissolve the discomfort of a hard call by giving the call away, and the discomfort didn't vanish — it became "wait, who's actually in charge here?", broadcast to your boss.
>
> And notice the compounding. If you traded for an advisor relationship in D2, Marcus holds you to it out loud — the deal was *early voice, not the wheel*. If you abdicated in D2 *as well*, this is now a pattern Sam has to step in and fix in real time. If you planted a firm flag back in D1 and then abdicate now, Sam sees the whiplash and flags the inconsistency. **AID**: the *action* was handing off your own decision to the most comfortable expert; the *impact* is a confused team, a senior IC forced to publicly refuse the wheel, and a manager who now isn't sure what you do; the *do-differently* is to credit the expertise *while keeping the pen* — "Marcus, your read carries the most weight here, so make the strongest case and I'll make the call." That gives him 100% of the influence and you 100% of the role. The trust was free and right. Giving away the job was expensive and wrong. You only needed the first.

---

### Summary Content — Performance Review

**Dominant style** = the mode of `choiceHistory` across the three decisions. Ties broken by frequency in the order **Diplomat > Negotiator > Fighter > Avoider > People Pleaser** (same as Scenario 1). The two lines below feed §2.3's "What it cost you" / "What it earned you," written in Future You's voice.

```ts
summary: {
  costsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
  earningsByDominantStyle: {
    fighter: "...",
    negotiator: "...",
    diplomat: "...",
    avoider: "...",
    'people-pleaser': "...",
  },
}
```

#### 🔥 Fighter — dominant

**What it cost you:**
> You spent your first weeks as lead proving you outrank a man who never once disputed it. Every time Marcus offered to *align*, you answered with the org chart — and the most experienced designer on your team slowly stopped lending you the eight years and started just executing the decided thing, factually, coldly, no air cover when you needed it in front of Sam. That's the **Compete** quadrant's tax: you win every exchange about who's in charge and lose the thing you became lead *for*, which was his best thinking. A senior IC who goes quiet isn't pacified. He's gone, he just hasn't told HR yet.

**What it earned you:**
> Nobody on this team wonders whether you'll fold under pressure or get rolled by tenure, and for a new lead facing a report with eight more years, that clarity is worth real money — you planted the flag and it stayed planted. When you finally pointed the directness at the *work* instead of at Marcus, you ran the most honest crit Sam had seen in years. The whole job now is keeping the spine and adding back the half of **Radical Candor** you keep skipping: care personally. Authority you have to keep *re-proving* isn't authority — it's anxiety in a louder voice. Use the spine to make the room safe enough that your most experienced person fights *beside* you, not just under you.

#### ⚖️ Negotiator — dominant

**What it cost you:**
> You turned a relationship into a series of clean deals, and Marcus noticed every one. The advisor role, the design-off, the "headline now, decision tomorrow" — all genuinely good *mechanisms*, all quietly teaching your most senior report that influence, a voice, and respect are things he *bargains for* rather than things you extend because he earned them. **SBI**: the trades were sound, but a few of them spent trust to buy alignment, and with a report, trust is the currency you were supposed to be *building*, not spending. He'd have traded a little of your efficiency for the sense that you two were ever just *thinking together*, no terms on the table.

**What it earned you:**
> You're the lead who finds the structure when everyone else is stuck in a standoff — you took a public challenge from a more senior designer and converted it into fair process instead of a power fight, repeatedly, and Sam clocked it as someone who runs a room well. That's the **Compromise** quadrant working as designed: outcomes over ego, dignity intact on both sides. Carry it. Just remember that some moments with your own people don't want a deal — they want you to *give* the thing (the early voice, the respect, the "you're right") freely, and ask for nothing back. The deliverable was always great. The free sentence is the one you kept skipping.

#### 🤝 Diplomat — dominant

**What it cost you:**
> Time, and a sliver of the decisiveness a team also needs to feel from a brand-new lead. Naming your own insecurity, handing Marcus the pen, repairing in private before performing in public — all the right calls. But **Collaborate** is the most expensive quadrant to run, and the failure mode hiding inside your strength is the day you have to make a call Marcus genuinely hates and *don't* fold. Safety isn't agreement. If "let's decide together" ever hardens into "every call gets re-opened until everyone's comfortable," you'll have traded being the lead for being the facilitator — and a team with eight years of seniority in the room will, kindly, start deciding without you.

**What it earned you:**
> The rarest thing on any design team: a senior IC with eight more years than you who will say "I was wrong" out loud, in front of his manager and yours, because you built a room where that's just how you get to the right answer. You did it in three weeks, and you did it by refusing the two easy flinches — you didn't defer to his tenure and you didn't assert over it; you made seniority irrelevant and *evidence* the currency. You named the eight years that scared you instead of letting them leak out as control, and Marcus decided to give you his actual best thinking. That's the entire reason to want a report more experienced than you. The best move in the highest-stakes room was set up two weeks earlier, in a 1-on-1 nobody saw. That's the whole game.

#### 🚪 Avoider — dominant

**What it cost you:**
> A reputation you didn't mean to build, in front of the one person whose read of you matters most right now. One "let me sit with it" is prudence; you reached for it enough times on the same problem that Sam started wondering — out loud, to you — whether the new lead can actually *decide in the room*, which is precisely where a lead is supposed to be able to. The **Avoid** quadrant is a loan against your credibility and you carried a balance, with screen 2 dead-ending real users in prod the whole time. Worse: every vacuum you left, Marcus quietly filled — attaching the deadlines, writing the recommendation, doing the deciding you kept deferring, with less authority and more frustration than either of you needed.

**What it earned you:**
> You almost never made a charged moment worse by reacting hot in front of the team — and as a new lead who'd just been publicly challenged, the instinct *not* to fire back five minutes later was genuine maturity, not weakness. Refusing to debug live data on the fly, or to redesign a direction in the heat of a crit, kept you from saying things you couldn't take back. The fix is small and fully in reach: put a *clock* on every defer. "Thursday AM, after I review his two old flows" is delegation; "let me sit with it" is the thing Sam started worrying about. A specific defer is leadership. An unspecific one, to a team waiting on you, reads as the decision quietly evaporating — and your most experienced report filling the gap.

#### 🙏 People Pleaser — dominant

**What it cost you:**
> You kept trying to make Marcus comfortable by giving away the one thing he actually needed you to keep: the role. "You should've gotten this job," "let's keep everything the same," "you run screen 2" — each one felt like humility and landed as a vacuum, and vacuums on a design team get filled by whoever's most confident in the room, which by default was the man with eight more years. Even *Marcus* kept pushing the authority back at you, because a leaderless team makes *his* job harder too — he wanted a boss, not a fan. **Accommodate** is the quadrant where you sacrifice your own position to dissolve discomfort, and the discomfort didn't disappear; it spread to the whole team as "wait, who's actually deciding?" — eventually loud enough that Sam had to step in and answer it for you.

**What it earned you:**
> Disarming, genuine humility — and in a brand-new lead facing a report with twice the tenure, humility is rare and it's the raw material of someone people will follow. You never once let ego pick a fight, and your instinct to credit Marcus's expertise was *right*; the team felt the absence of arrogance and it mattered. Don't kill that instinct — aim it. The lesson the weeks kept teaching is that *crediting the expertise* and *keeping the pen* are two different acts: "you're right about screen 2, let's fix that one, the rest holds" gives Marcus the full win he actually asked for and keeps you the lead. **AID** in a line: keep the warmth, separate it from the abdication, and give people respect with a sentence that doesn't give away your job. The trust was free and true. The wheel was never yours to hand over.

---

## Notes for the Code Agent — Scenario 3 (`#crit-aftermath`)

This scenario uses the **same optional, backwards-compatible type additions** introduced for Scenario 1 (the `warmup?` field on `Decision`, `incomingVariants?` on `Decision`, and `lockedWhen` / `lockedReason` / `conditionalChatItems` / `conditionalDM` on `ResponseOption`). No new types are required. Everything below is what's specific to Scenario 3.

### (a) Cast & avatar colors (NEW characters)

All three brand colors below are distinct from the existing cast (Devon `#FBBC04`, Maya `#34A853`, Sam `#5F6368`, Priya `#EA4335`, Riya `#4285F4`, You `#202124`, Future You `#9333EA`).

| Speaker | Initials | `avatarColor` | Role / notes |
|---|---|---|---|
| **Marcus** | `MR` | `#00897B` (Material Teal 600) | Staff designer, 8 yrs your senior, now your report. The senior dissenter. Sentence-case, full punctuation, measured. Principled, prickly-but-fair, **not a villain** — he concedes the timing, brings receipts, and even owns his own miss in D3. |
| **Tara** | `TA` | `#3F51B5` (Material Indigo 500) | Mid-level designer, your report, witnessed the crit. The team's bellwether — when she's confused, the team is confused. Lowercase, emoji, warm, a little nervous. |
| **Lena** | `LN` | `#FF7043` (Material Deep Orange 400) | Marcus's work bestie / confidant. **DM recipient only** — never appears in-channel. This is who the player intercepts Marcus's DMs *to* (the S3 analogue of Maya→Riya). |
| **Sam** | `SM` | `#5F6368` (existing) | Your manager. Appears in-channel in **D3** as the observing boss, and in some D1/D2 paths. Existing avatar, brief and weary voice. |
| **You** | `AY` | `#202124` (existing) | Right-aligned. The new lead, 3 weeks in. |
| **Future You** | `AY+` | `#9333EA` (existing) | Coaching DMs only. |

> **Accent color:** scenario `accentColor` is `#EA4335` (Google Red) per the brief. This is the scenario's stripe/theme color and is unrelated to per-character avatar colors. Note it coincides with Priya's avatar color, but **Priya does not appear in Scenario 3 at all** (neither Director-Priya nor PM-friend-Priya), so there's no collision on any surface.

### (b) Decision 2 — `incomingVariants` keyed to `choiceHistory[0]`

D2 has **one shared decision**. Only the **first incoming message** changes based on the D1 choice; shared message #2 ("how do you want this to work between us?") and all five options are identical across paths. Render the variant **first**, then shared #2 — identical mechanic to Scenario 1's D2.

| `choiceHistory[0]` | Marcus's posture on entry | variant id |
|---|---|---|
| `fighter` | entrenched / formal — "I'll keep feedback in writing from here on out" | `1a` |
| `negotiator` | competitive / proving-it — "who's the judge if there's no test?" | `1b` |
| `diplomat` | warmed / collaborative — "yesterday was good... I owe you something" | `1c` |
| `avoider` | patient-but-stalled — "I still don't have a direction and the milestone's Monday" | `1d` |
| `people-pleaser` | uncomfortable / over-given-to — "you handed me the whole direction... weird spot" | `1e` |

> **Timestamps:** all five variants stamped **9:40 AM Wednesday**, shared #2 stamped **9:41 AM** ("[+1 min from variant]"). Unlike Scenario 1 (where D1 paths ended at wildly different times), Scenario 3's D1 is a single in-channel afternoon that converges onto a next-morning 1-on-1 regardless of path, so a uniform D2 timestamp is correct and intentional.

### (c) Decision 3 — lock conditions + compounding modifiers

D3's `incoming` is **fully shared** (no variants). Branching lives **per option** via lock conditions and `conditionalChatItems` / `conditionalDM` blocks, exactly per Scenario 1's D3 assembly rule:
> 1. Render base `chatItems`. 2. Append each `conditionalChatItems` block in array order where `when()` is true (multiple can stack). 3. For DMs, the **first** matching `conditionalDM` block supplies the panel; else base `dmIntercepted`. (`[otherwise]` DM = base.) 4. `futureYou` is static per option.

**Lock conditions (2 meaningful locks across the five options):**

| Option | `lockedWhen` (choiceHistory) | Rationale |
|---|---|---|
| 🔥 Fighter | *(never)* | Calling it out in the open is always available; the room's *read* shifts via conditionals. |
| ⚖️ Negotiator | *(never)* | Always selectable; framing shifts (alignment vs. transaction) via conditionals. |
| 🤝 Diplomat | `choiceHistory[1] === 'fighter' \|\| choiceHistory[1] === 'people-pleaser'` | "Exactly the room I want this to be" is only true if you built the room in D2. Rank-pulling (fighter) or abdication (people-pleaser) in the 1-on-1 means the trust was never banked. Locked-reason copy is in the option block. |
| 🚪 Avoider | *(never)* | Always selectable; the room's patience (rigor vs. flinch) shifts via conditionals + Sam's presence. |
| 🙏 People Pleaser | `choiceHistory[1] === 'diplomat' \|\| choiceHistory[1] === 'fighter'` | Handing Marcus the wheel now visibly contradicts a D2 where you either co-built the relationship as the lead (diplomat) or explicitly claimed the lead role (fighter) — a witnessed contradiction with Sam present. Locked-reason copy is in the option block. |

> **Net effect (intended Bandersnatch lock):** D3's two locks read **`choiceHistory[1]` (the D2 choice)**, not `[0]`. A D2 of `diplomat` locks the People Pleaser reflex (you can't abdicate after partnering as the lead). A D2 of `fighter` *also* locks People Pleaser (you can't abdicate after planting the authority flag) **and** locks Diplomat (you can't claim "the room I want" after rank-pulling). A D2 of `people-pleaser` locks Diplomat (no trust banked). **Edge-case safety:** in **every** D2 path, at least Fighter, Negotiator, and Avoider remain selectable, so D3 is never a dead end. Quick check by D2 choice: `fighter` → Fighter/Negotiator/Avoider open (Diplomat+PP locked); `people-pleaser` → Fighter/Negotiator/Avoider/PP open (Diplomat locked); `diplomat` → Fighter/Negotiator/Avoider/Diplomat open (PP locked); `negotiator`/`avoider` → **all five open**.

**Compounding modifier index (which prior choice each option's conditionals read):**

| Option | reads `choiceHistory[0]` (D1) for… | reads `choiceHistory[1]` (D2) for… |
|---|---|---|
| 🔥 Fighter | "second hot crit / pattern" Sam line (`[0]=fighter`) | Marcus publicly owns his miss + Sam praise (`[1]=diplomat`) vs. cold/no-air-cover + Sam notes formality (`[1]=fighter`/`people-pleaser`) |
| ⚖️ Negotiator | team wary of "offline" (`[0]=avoider`) | deal lands as alignment (`[1]=diplomat`) vs. Marcus names the deal-pattern (`[1]=negotiator`) |
| 🤝 Diplomat | Tara "different room" reflection (`[0]=diplomat`) / Sam "course-corrected" recovery (`[0]=fighter`) | *(gate — see lock)*; DM warmth: Marcus all-in (`[1]=diplomat`) vs. still-guarded (`[otherwise]`) |
| 🚪 Avoider | Tara + Sam "second offline on screen 2" (`[0]=avoider`) | Marcus fills vacuum w/ deadline (`[1]=avoider`) / shortcuts it for you (`[1]=diplomat`); Sam DM fires if `[0]=avoider OR [1]=avoider` |
| 🙏 People Pleaser | Sam whiplash line if you planted a flag in D1 then abdicate (`[0]=diplomat OR fighter`) | *(gate for the lock)*; Sam steps into vacuum (`[1]=people-pleaser`) / Marcus invokes the deal (`[1]=negotiator`); DM: Marcus-to-Lena if `[1]=negotiator`, else Sam-to-you |

### (d) No Priya in this scenario

Unlike Scenario 1, **neither Priya appears** in Scenario 3 — not in-channel, not in DMs. The DM-intercepted recipient is **Lena** (Marcus's confidant). The in-channel manager is **Sam**. Avoid reusing the Priya avatar here.

### (e) DM-intercepted sources by option (for panel wiring)

Most intercepted DMs are **Marcus → Lena** (the dissenter venting to his confidant — the scenario's signature interception). Three blocks intercept a **Sam → (you)** *coaching/observation* DM instead (D3 Fighter `[1]=diplomat`, D3 Avoider `[0]/[1]=avoider`, D3 People Pleaser `[otherwise]`) — these are the manager privately reacting, which raises the stakes by showing the player their boss is forming a read. The panel already supports arbitrary `from`/`to`, so no change needed; just note that `to: "(you)"` renders as a DM *to the player* (Future-You-card-adjacent in tone but it's Sam, not Future You — keep it in the DM-intercepted card, not the Future You card).

### (f) Timeline continuity

- **D1:** Tuesday, 3:05–3:11 PM, in `#crit-aftermath`, immediately after the crit. Single in-channel beat; all five options resolve the same afternoon (3:13–3:19 PM range).
- **D2:** Wednesday, 9:40–9:47 AM, 1-on-1 DM with Marcus. All paths converge here regardless of D1 (uniform timestamps — see (b)).
- **D3:** **Tuesday the following week**, 3:02 PM, back in `#crit-aftermath` after the next crit, with Sam observing in-channel. One week of "how you worked with Marcus" sits between D2 and D3, which is exactly the gap the compounding modifiers dramatize. D3 consequences run 3:08–3:13 PM.
- The Monday milestone referenced throughout falls **between D2 (Wed) and D3 (next Tue)** — screen 2 shipped over that weekend, which is why D3 has *real session data* and the stakes are now live-in-prod, not hypothetical.

---

## Notes for the Code Agent

- **All content is static.** Bake the full scenarios into `data/scenarios.ts`. No runtime fetching, no API calls.
- **Time markers are first-class chat items**, not styling. They appear in the `chatItems` array as `{ type: 'time-marker', label: '...' }`.
- **The DM-intercepted card supports multiple DMs.** Diplomat path in Scenario 1 Decision 1 shows two DMs side-by-side. Build the panel to handle 1+ DMs.
- **Player's response is appended to the chat history** as a right-aligned message with the player's avatar before the consequence chat items stream in.
- **State to track during a scenario:**
  - `currentDecisionIndex` (0, 1, 2)
  - `choiceHistory: ConflictStyle[]` — accumulates as player makes choices
  - `chatLog: ChatItem[]` — full message history for the current scenario
- **Dominant style computation for the summary:** the mode of `choiceHistory`. Ties broken by frequency in the order: Diplomat > Negotiator > Fighter > Avoider > People Pleaser.

End of document. Will append remaining scenarios as written.
