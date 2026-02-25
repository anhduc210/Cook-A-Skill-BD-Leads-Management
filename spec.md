BD Agent — Spec  
Owner: Dylan  
Primary Users: BD / Partnerships / Founder  
Goal: From a campaign brief, automatically **discover, score, and draft outreach** to relevant leads (Telegram‑first), running as a single Claude Skill.

***

## 1) Objective

From one campaign brief, the BD Agent will:

- **Auto‑discover leads (default path)**  
  - Use web search to find 20–50 relevant leads (e.g., “crypto KOLs talking about Polymarket”). [crawleyagent](https://crawleyagent.com)
  - Extract topics, role, language, and personality from public text.  

- **Score & tier leads**  
  - Assign Relevance / Influence / Fit scores with clear rubric.  
  - Tier A/B/C with short justification per lead. [weezly](https://weezly.com/blog/ai-lead-scoring-boost-outbound-sales-roi/)

- **Draft personalized Telegram DMs**  
  - Generate tailored outreach drafts for A & B leads using real signals from their content. [docsbot](https://docsbot.ai/prompts/business/personalized-dm-generator)

Manual lead list input is **fallback only** (rare case when user wants to force certain leads in).

***

## 2) Problem Statement

Today, BD campaigns require:

- Manually hunting for KOLs / partners across X, Telegram, blogs.  
- Inconsistent gut‑feel scoring and scattered notes.  
- Generic cold DMs not tailored to each person’s interests or style.

BD Agent turns this into an automated loop: **brief → auto‑research → qualified list → personalized DMs**, without needing custom code or a CRM.

***

## 3) Business Value

- **Zero‑setup lead generation**: user writes a campaign brief; skill finds people to talk to. No CRM, no VA, no custom code — a single Claude conversation replaces the entire top-of-funnel research workflow.
- **Consistent qualification**: transparent scoring and tiers across campaigns — same rubric every time, no gut-feel drift.
- **Higher reply odds**: personalized outreach grounded in what each lead actually talks about. Personalized cold DMs typically see 3–5× higher reply rates than generic templates.
- **Reusable playbook**: the Markdown output can be handed to any BD teammate, used as a briefing doc, or rerun for a new campaign with a different brief.
- **Radical time savings**: replaces 2–3 hours of manual KOL research + drafting per campaign. Full loop (discover → score → draft) runs in a single session with zero external infrastructure — no APIs, no database, no integrations required.

***

## 4) Scope

### 4.1 In Scope (MVP Skill)

- Input:  
  - Campaign brief (required).  
  - Optional: additional “must‑include” leads (rare case).

- Processing:  
  - Auto‑discover most leads via web search.  
  - Score, tier, and summarize each lead.  
  - Draft Telegram‑style DMs for Tier A/B.

- Output:  
  - One Markdown report: summary + lead table + per‑lead DM drafts.

### 4.2 Out of Scope

- Sending messages via Telegram API.  
- Persistent database or cross‑run memory.  
- Multi‑channel outreach (X / LinkedIn / email).  

***

## 5) Input Contract

### 5.1 Campaign Brief (required)

From a short text block, the skill extracts:

- `campaign_name`  
- `product_description`  
- `target_audience` (ICP)  
- `value_proposition`  
- `key_topics` (e.g., Polymarket, prediction markets, crypto KOLs)  
- `target_languages` (EN / VN / mixed)  
- `campaign_tone` (casual, professional, community, etc.)  
- `campaign_goal` (awareness, collab, beta testers…)

Missing fields → marked `MISSING` in report; used as warning.

### 5.2 Optional Manual Leads (fallback)

Only used in edge cases (e.g., “always include these 3 partners”):

```text
---
name: Alice
handle: @alice_kol
profile_url: https://t.me/alice
notes: "Must include — existing contact."
---
```

These leads are merged into the discovered list and scored like others, but labelled `[User‑provided]`.

***

## 6) Clarifying Questions (Required)

Before discovery or scoring, the skill must ask:

1. **Q1 — Desired action**  
   “What do you want these leads to do?”  
   *(e.g., reply with an opinion, join a call, test a feature, post content)*

2. **Q2 — Tone**  
   “How should the DM feel?”  
   *(e.g., founder‑to‑founder, casual degen, professional, researcher)*

3. **Q3 — Hard filters**  
   “Any strict filters?”  
   *(e.g., only EN posts, exclude meme‑coin shillers, min follower size, region)*

If any question is unanswered → stop and re‑ask; do not continue.

***

## 7) Lead Discovery & Scoring

### 7.1 Discovery

- Build search queries from:
  - ICP + `key_topics` + campaign goal.  
  - Examples: `"Polymarket prediction market KOL"`, `"crypto influencer prediction markets opinion"`, etc. [graphlinq](https://graphlinq.io/blog-posts/smarter-bets-on-polymarket-how-graphai-agents-give-you-the-edge)

- Use `web_search` to retrieve candidate profiles / content (X threads, blogs, Telegram channel bios, etc.).  
- For each candidate:
  - Extract:
    - Name / handle / profile URL.  
    - Short description/bio.  
    - Key topics they talk about.  
    - Language and rough tone.

### 7.2 Scoring Rubric (0–30 total)

| Dimension | Range | Basis |
|---|---|---|
| **Relevance** | 0–10 | Topic overlap with `key_topics` + ICP (see brackets below) |
| **Influence** | 0–10 | Follower count or role prominence (see brackets below) |
| **Fit** | 0–10 | Voice/tone alignment with `campaign_tone` + goal (see brackets below) |

**Relevance Score Brackets:**

| Signal | Score |
|---|---|
| Content revolves entirely around the topic; deep insights, consistent coverage | 9–10 |
| Frequent mentions, clear sustained interest | 6–8 |
| Occasional or passing mentions | 3–5 |
| Little to no mention of relevant topics | 0–2 |

**Influence Score Brackets:**

| Estimated Followers / Reach | Score |
|---|---|
| > 500K | 10 |
| 100K – 500K | 8–9 |
| 50K – 100K | 6–7 |
| 10K – 50K | 4–5 |
| 1K – 10K | 2–3 |
| < 1K or Unknown | 1 |

> Role prominence (e.g., confirmed VC partner, protocol co‑founder, known CT figure) → apply **+1 adjustment** to bracket score, max 10.
> If no follower data is available and only role prominence is known → assign bracket floor score (1) and mark as `Influence: estimated`. Never fabricate or guess a follower count.

**Fit Score Brackets:**

| Signal | Score |
|---|---|
| Perfect match — language, style, and audience all align with campaign tone | 9–10 |
| Mostly aligned — minor DM text adjustment needed | 5–8 |
| Mismatched language or vibe — requires significant rewrite or skip | 0–4 |

Tier mapping:

- Tier A: 23–30 → priority.
- Tier B: 16–22 → secondary.
- Tier C: ≤15 → low priority.

Each dimension must include a short explanation referencing visible signals (e.g., “Weekly Polymarket threads”, “newsletter on prediction markets”).

***

## 8) Outreach Drafting (Telegram Style)

For **Tier A & B** leads:

- Build DM with 3 parts:

  1. **Hook** — reference a real topic or angle from their content.  
  2. **Bridge** — 1–2 lines connecting your campaign to that topic.  
  3. **Soft CTA** — aligned with Q1 (e.g., “open to a quick take?”).

- Personalization:

  - Use their name/handle.  
  - Mention 1–2 specific themes (not fabricated posts).  
  - Adapt tone based on discovered personality (degen / serious / educator).

- Guardrails:

  - No invented evidence (“I loved your post about X” if no such post).
  - Never fabricate follower counts or engagement metrics — only use data explicitly returned by web search.
  - Keep messages short (2–4 sentences). **Hard limit: ≤ 300 characters total.** Telegram DMs beyond this length have significantly lower read rates; trim aggressively if needed.
  - If signals are weak → label `[GENERIC — manual edit recommended]`.

Tier C: no DM generated (only noted in table as “skip / nurture later”).

***

## 9) Output Structure (Markdown)

### 9.1 Executive Summary

- Campaign recap (goal, ICP, topics, tone).  
- Lead stats: total, count per tier, any geographic/language skew.  
- 3–5 bullets with key insights (e.g., “Most Polymarket KOLs lean degen and prefer casual tone.”).

### 9.2 Lead Scorecard (Table)

| Name / Handle | Tier | Score (R/I/F) | Topics | Tone & Personality | Notes |
|---|---|---|---|---|---|
| @alice_kol | 🟢 A | 27 (9/8/10) | Polymarket, prediction markets | Casual, data‑driven | Writes weekly PM threads |
| @bob_defi | 🟡 B | 19 (7/6/6) | DeFi, some PM | More serious | Mentioned PM twice |

### 9.3 Lead Insights (Optional per‑lead block)

For top A leads, short paragraphs summarizing why they’re priority.

### 9.4 Telegram DM Drafts

Grouped by tier:

```md
## Tier A — Telegram DM Drafts

### @alice_kol
> Hey Alice, I’ve seen your threads breaking down Polymarket markets — especially your take on how prediction markets can surface information faster than CT. We’re experimenting with a new agent‑driven Polymarket tool and would love your quick take. Would you be open to a short chat this week?

[Personalization source: Polymarket analysis threads, prediction market angle]

## Tier B — Telegram DM Drafts
...
```

***

## 10) Edge Cases

- **Too few leads found**
  - Return whatever number is reliable; explicitly suggest broader queries or adding manual “must‑include” leads.

- **Conflicting or thin data**
  - Mark follower/role info as `Unknown` or `CONFLICT` and adjust Influence score conservatively.

- **Non‑matching language**
  - If content language doesn’t match `target_languages`, reduce Fit and flag in notes (“EN only; campaign target is VN”).

- **All leads score Tier C (≤15)**
  - Do not generate any DMs. Output the scorecard and explicitly flag: “No Tier A or B leads found — suggest broadening `key_topics`, relaxing hard filters, or providing manual must-include leads.”

- **Fabricated or unverifiable data**
  - If follower count, bio, or topic signals cannot be confirmed from search results → mark as `Unknown`, do not estimate, and score conservatively. Never invent data to complete a score.

***

## 11) MVP Success Criteria

- With only a campaign brief, the skill:
  - Finds at least 10–20 meaningful leads for a focused niche.
  - Assigns reasonable A/B/C tiers with clear explanation.
  - Produces Telegram‑ready DM drafts for all A/B leads. Each DM ≤ 300 characters.
- All personalization is based on real, discoverable content; no hallucinated references.

### 11.1 Pre‑Demo: web_search Discovery Validation (Day 2 gate)

`web_search` is the core engine of this skill — if it can't return usable profiles, nothing else works. Before the demo, run a **discovery smoke test** across at least 3 different niches:

| Test Niche | Sample Query | Pass Condition |
|---|---|---|
| Crypto KOLs | `"Polymarket prediction market influencer crypto Twitter"` | ≥ 5 scoreable profiles returned |
| DeFi builders | `"DeFi protocol founder newsletter prediction markets"` | ≥ 5 scoreable profiles returned |
| Alternative niche (your choice) | e.g., `"AI agent developer crypto opinion"` | ≥ 5 scoreable profiles returned |

If any niche returns < 5 scoreable results → flag immediately and adjust search query construction logic before demo day.

***

## 12) Full Sample Output

Below is a complete example of what the skill produces from a single campaign brief. Use this as a reference when evaluating output quality.

---

**Campaign brief used:**
> Product: PredictDAO — prediction market aggregator. Target: crypto KOLs who write about Polymarket or prediction markets. Goal: get 5–10 people to try the beta and post a short take. Tone: founder-to-founder, casual.

---

### 12.1 Executive Summary

**Campaign:** PredictDAO Beta Outreach
**Goal:** Beta testers who will post a take
**ICP:** Crypto KOLs interested in prediction markets
**Tone:** Founder-to-founder, casual
**Topics:** Polymarket, prediction markets, crypto opinion

**Lead stats:** 18 total discovered · 4 Tier A · 7 Tier B · 7 Tier C
**Language skew:** 80% EN, 20% mixed EN/VN

**Key insights:**
- Most Polymarket KOLs lean degen and prefer casual, data-driven communication.
- VN-language accounts have lower follower counts but higher engagement ratios — worth targeting for regional growth.
- Several Tier B leads post infrequently but have niche authority; could be upgraded if engagement data is confirmed.
- No obvious spam/shill accounts detected in top results.

---

### 12.2 Lead Scorecard

| Name / Handle | Tier | Score (R/I/F) | Topics | Tone & Personality | Notes |
|---|---|---|---|---|---|
| @alice_kol | 🟢 A | 27 (9/9/9) | Polymarket, prediction markets, CT takes | Casual, data-driven | Weekly PM threads; 120K followers |
| @bob_defi | 🟢 A | 24 (8/8/8) | DeFi, Polymarket, market structure | Serious but accessible | Newsletter with 40K subscribers |
| @carol_trader | 🟡 B | 20 (8/6/6) | Crypto trading, some prediction markets | Degen, opinionated | 12K followers; mentioned PM twice |
| @dave_research | 🟡 B | 17 (7/5/5) | Research, macro, occasional PM | Academic tone | Low followers but cited by larger accounts |
| @eve_anon | 🔴 C | 12 (5/4/3) | Meme coins, general crypto | Meme-heavy | Mostly off-topic; skip |

---

### 12.3 Telegram DM Drafts

#### Tier A

**@alice_kol**
> Hey Alice — love how you break down Polymarket odds vs. CT sentiment. Building an aggregator that surfaces that gap automatically. Would you be open to trying the beta and sharing a quick take?

*(~230 chars · Personalization: PM threads + data-driven angle)*

---

**@bob_defi**
> Hey Bob — your newsletter take on market structure in prediction markets was sharp. We're building PredictDAO to surface those inefficiencies. Would love your beta feedback — open to a quick look?

*(~240 chars · Personalization: newsletter, market structure angle)*

---

#### Tier B

**@carol_trader**
> Hey Carol — saw you mentioned Polymarket a couple times. We're running a beta for a PM aggregator and looking for sharp traders to stress-test it. Interested?

*(~190 chars · Personalization: PM mentions + trader framing)*

---

**@dave_research** `[GENERIC — manual edit recommended]`
> Hey Dave — your research on macro and prediction markets caught our eye. We're building PredictDAO and would love a researcher's perspective on the beta. Open to a look?

*(~205 chars · Signals thin — verify content before sending)*
