# Design Specification

This document defines the visual identity, screen layouts, components, and interactions for `v3_FINAL_actually_final.fig`. The visual language is **Google Material Design**, applied to a parody Google Workspace setting (Shoogle Chat). Use this document alongside `PLANNING.md` (architecture) and `CONVERSATIONS.md` (content).

---

## 1. Visual Identity

### 1.1 Color Tokens

| Token | Hex | Use |
|---|---|---|
| `--bg-canvas` | `#FFFFFF` | Main background |
| `--grid-dot` | `#F1F3F4` | Dot grid pattern |
| `--surface` | `#FFFFFF` | Cards (with shadow) |
| `--surface-hover` | `#F8F9FA` | Hover state on cards |
| `--border-subtle` | `#DADCE0` | Dividers, card edges |
| `--text-primary` | `#202124` | Body, headings |
| `--text-secondary` | `#5F6368` | Timestamps, captions |
| `--accent-fighter` | `#EA4335` | Google Red — Fighter option |
| `--accent-negotiator` | `#FBBC04` | Google Yellow — Negotiator option |
| `--accent-diplomat` | `#34A853` | Google Green — Diplomat option |
| `--accent-avoider` | `#5F6368` | Cool Grey — Avoider option |
| `--accent-people-pleaser` | `#4285F4` | Google Blue — People Pleaser option |
| `--accent-future-you` | `#9333EA` | Material Purple — Future You accent |

### 1.2 Background Pattern

Light grey dot grid: dots at `#F1F3F4`, **20px apart**, **1.5px diameter**.
- **Landing screen:** full opacity, visible across the whole canvas.
- **Scenario thread screens:** 40% opacity, subtle background texture.

### 1.3 Typography

| Use | Font | Source |
|---|---|---|
| Display + headings | `"Google Sans", "Roboto", sans-serif` | Google Fonts |
| Body | `"Roboto", sans-serif` | Google Fonts |
| Monospace (title, timestamps, file names) | `"Roboto Mono", monospace` | Google Fonts |

### 1.4 Type Scale

| Style | Size | Line height | Weight |
|---|---|---|---|
| Display | 48px | 56px | 500 |
| H1 | 32px | 40px | 500 |
| H2 | 24px | 32px | 500 |
| H3 | 20px | 28px | 500 |
| Body L | 16px | 24px | 400 |
| Body | 14px | 20px | 400 |
| Caption | 12px | 16px | 400 |

### 1.5 Spacing & Elevation

- **Card padding:** 24px
- **Message padding:** 16px
- **Card border-radius:** 8px
- **Message bubble border-radius:** 16px
- **Small chip border-radius:** 4px

**Elevation 1 (cards, idle):**
`box-shadow: 0 1px 2px rgba(60,64,67,0.3), 0 1px 3px 1px rgba(60,64,67,0.15)`

**Elevation 2 (cards, hover):**
`box-shadow: 0 1px 3px rgba(60,64,67,0.3), 0 4px 8px 3px rgba(60,64,67,0.15)`

### 1.6 Motion

- All transitions: **200–300ms ease-out**.
- **Slide-in panel** from right edge: 320ms ease-out.
- **Message arrival:** fade-in + 8px translate-y, 200ms ease-out.
- **Typing indicator:** three-dot bouncing animation, 1.4s loop.
- **Hover on cards:** elevation bump + 1.02 scale, 200ms ease-out.

---

## 2. Screen Specifications

### 2.1 Landing Screen (`/`)

**Background:** white canvas, full dot grid visible.

**Top bar (sticky, transparent):**
- Top-left: small **Shoogle** wordmark (text only, "Shoogle" with the two `o` letters in Google Red `#EA4335` and Google Green `#34A853` to mimic the Google wordmark).
- Top-right: fake user avatar (32x32 circle with initials "AY"), small status dot `🟢` next to it labeled "Active."

**Hero block (centered, 96px from top):**
- **Display heading**, monospace: `v3_FINAL_actually_final.fig`
- **Subtitle**, Body L: *a lateral leadership simulator*
- **Caption**, Body: *3 scenarios. 5 conflict styles. 1 honest performance review.*

**Scenario grid (below hero, 32px gap):**
3 cards in a row. Responsive: 1 column on screens narrower than 768px.

Each card (see Component Spec §3.1):
- Channel name in mono
- Preview line
- Difficulty chip
- Unread badge
- Accent stripe on top

**Footer:** centered, Caption text in `--text-secondary`:
*Built for Leadership by Design · CCA · 2026*

### 2.2 Scenario Thread (`/scenario/[id]`)

**Layout:** three columns
- **Left sidebar (240px):** Shoogle Chat decorative sidebar
- **Main column (flex-1):** chat thread
- **Right panel (380px, slide-in):** consequence panel

**Left sidebar contents (non-functional, decorative):**
- Workspace label "Shoogle" with the colored wordmark treatment.
- "Channels" list: `#general`, `#design-team`, `#launches`, `#dogs-of-shoogle`, plus the three scenario channels. Active channel highlighted with `--surface-hover` background and a 3px left border in `--accent-people-pleaser`.
- "Direct messages" list: Devon, Maya, Sam, Future You (with purple avatar).
- These items don't navigate anywhere when clicked. Pure atmosphere.

**Main column:**
- **Top bar:** channel name in mono, channel topic in Caption italic underneath, 1px bottom border in `--border-subtle`.
- **Chat history:** scrollable, messages stream in (Component Spec §3.2).
- **Bottom area:** response options appear after each incoming decision message (Component Spec §3.4).

**Right slide-in panel:**
- Closed by default. Slides in from the right after the player makes a decision and the in-thread consequences have streamed.
- Contains: DM intercepted card (Component Spec §3.6) + Future You DM card (Component Spec §3.7).
- Has a **"Continue →"** button at the bottom to advance to the next decision.

### 2.3 End-of-Scenario Summary (`/scenario/[id]/summary`)

Full-screen card, max-width 640px, centered on dot-grid background.

**Header:** *"End of scenario — `#just-a-quick-ask 🙃`"* in H2.
**Subtitle:** *"Future You has filed your performance review."* in Caption.

**Style distribution bar:** horizontal 5-segment stacked chart showing the player's choice mix across the three decisions. Each segment uses its conflict-style accent color. Hover reveals the count (e.g., "Diplomat — 2 of 3").

**Three lines of text:**
- *Your dominant style this scenario:* `[computed]`
- *What it cost you:* `[narrative]`
- *What it earned you:* `[narrative]`

**Two buttons:**
- **Replay** — outlined, restarts the scenario
- **Next scenario →** — filled, routes to next scenario or back to landing if all done

---

## 3. Component Specifications

### 3.1 Scenario Card

- Surface: `--surface`, elevation 1
- Padding: 24px
- Border-radius: 8px
- **Top edge:** 3px solid stripe in scenario's accent color
- **Channel name:** mono, Body L, `--text-primary`
- **Preview line:** Body, `--text-secondary` (1 line, ellipsis on overflow)
- **Difficulty chip:** small pill, 4px border-radius, Caption text, with emoji (🔥 Familiar / 🔥🔥 Tense / 💀 Personal)
- **Unread badge:** 20x20 circle, accent color background, white number
- **Hover state:** elevation 2 + scale 1.02
- **Click:** routes to `/scenario/[id]`

### 3.2 Chat Message

- **Avatar:** 32x32 circle, colored background, white initial letter centered.
- **Header line:** `Name` in 14px weight 500, `Timestamp` in 12px secondary, 8px gap between.
- **Message body:** 14px Body, `--text-primary`, max-width 560px, wraps naturally.
- **Vertical spacing:** 16px between messages from different speakers, 4px between messages from the same speaker.
- **Player's own messages:** right-aligned, light blue background (`#E8F0FE`), 16px border-radius, 12px padding.

### 3.3 Typing Indicator

- Inline component appearing where the next message will land.
- Three dots in `--text-secondary`, animated with staggered bounce (200ms offset each).
- Shown for the duration of the message's `typingDelay` value (see CONVERSATIONS.md data structure).
- Replaced by the actual message when delay completes.

### 3.4 Response Option Button

- 5 buttons stacked vertically below the latest incoming message.
- Each button:
  - Full width within the message column (max 560px)
  - White surface, elevation 1
  - **Left edge:** 4px solid stripe in the conflict-style accent color
  - 16px padding
  - 8px border-radius
  - **Top row:** small chip with conflict-style emoji + label in 12px (e.g., "🔥 Fighter")
  - **Body:** the actual response text in 14px Body
- **Hover:** elevation 2 + scale 1.02 + slightly darker accent stripe
- **Click:**
  1. The selected response appears in the chat as a player message
  2. All five buttons disappear
  3. Typing indicator + consequence messages stream in
  4. Slide-in panel appears after all consequence messages are done

### 3.5 Time Marker

- Horizontal thin line across the chat column
- Centered text in mono, Caption size, `--text-secondary`
- Format: `— Friday, 8:47 AM —`
- 32px vertical margin above and below

### 3.6 DM Intercepted Card

- Within the slide-in panel, top section
- **Header chip:** 🛎️ icon + "DM intercepted" label
- **Subheader:** *From:* `[Devon]` *To:* `[Priya]` in Caption
- **Message body:** styled like a Shoogle Chat DM bubble — light grey background `#F1F3F4`, 16px border-radius, 14px Body text
- Subtle "this is what they're saying behind your back" energy

### 3.7 Future You DM Card

- Within the slide-in panel, below the DM intercepted card, separated by a 24px gap
- **Header chip:** 🔮 icon + "Future You" label
- **Avatar:** purple circle (`--accent-future-you`) with `AY+` (player initials + plus symbol)
- **Subheader:** "DM from Future You" in Caption
- **Message body:** styled with a left border in `--accent-future-you` (3px), 16px padding, 14px Body text
- Tone: kind but unflinching. Uses frameworks (Fighter/Negotiator/Diplomat/Avoider/People Pleaser, AID, SBI, Radical Candor, Thomas-Kilmann).

### 3.8 Performance Review Summary Card

- Full-screen modal
- Max-width 640px, centered
- Elevation 2
- Contains the 5-segment style chart, dominant style line, cost line, earnings line, and two action buttons (see §2.3)

---

## 4. Interaction Patterns

### 4.1 Message Streaming Sequence

For each incoming message:
1. Show typing indicator (Component §3.3) for `typingDelay` ms.
2. Replace typing indicator with the message (fade-in + translate-y).
3. Scroll chat to bottom.

### 4.2 Time Advance

When a time marker appears in the message sequence:
1. After previous message renders, wait 600ms.
2. Slide in the time marker line (fade-in, 300ms).
3. Wait 400ms.
4. Continue to next message.

### 4.3 Decision Flow

1. All incoming messages for the decision stream in.
2. Response options fade in from below, 8px translate-y, 200ms.
3. Player clicks an option.
4. Selected option's text appears as the player's own message (right-aligned bubble).
5. Other options disappear.
6. Consequence messages stream in using same rules as §4.1.
7. After all consequence messages render, wait 600ms.
8. Slide-in panel enters from the right (320ms ease-out).
9. Player reads DM intercepted + Future You.
10. Player clicks "Continue →" to advance to the next decision.

### 4.4 End-of-Scenario Trigger

After Decision 3's consequences and slide-in panel are done:
1. Player clicks "Continue →"
2. Route to `/scenario/[id]/summary`
3. Summary card fades in over dot-grid background.

---

## 5. Tone & Character Voices

Devon, Maya, and Future You are the three voices that carry the game. Character details below; full dialogue is in `CONVERSATIONS.md`.

### Devon — PM, peer

- Lowercase Slack typist.
- Uses 🙏 and 🙃 emojis often.
- Apologetic when asking for things, slightly passive about transmitting pressure ("leadership is breathing down my neck").
- Not a villain. Genuinely under stress.
- Dismisses small things ("nits") and asks big things in throwaway tone.

### Maya — Senior IC designer, your report

- Lowercase to start sentences, mostly proper punctuation.
- Hurt-but-trying-to-be-professional when sidelined.
- Direct when asking what happened.
- DMs her work bestie Riya when frustrated — those DMs are the ones the player intercepts.

### Future You — Coach, DMs only

- Plain sentences. No corporate hedging.
- Kind but unflinching.
- Names the choice. Names the cost. Offers a sharper version.
- References frameworks: Fighter/Negotiator/Diplomat/Avoider/People Pleaser, AID, SBI, Radical Candor, Thomas-Kilmann.
- Often closes with a question or a principle.
- Avoids preachy tone. Speaks like a slightly older you who's already made every mistake.

### Background characters

- **Riya** — Maya's work bestie, soft and supportive
- **Sam** — your manager, brief and slightly weary
- **Priya** — Devon's PM friend, gossipy and tired
