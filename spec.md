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

- **Zero‑setup lead generation**: user writes a campaign brief; skill finds people to talk to.  
- **Consistent qualification**: transparent scoring and tiers across campaigns.  
- **Higher reply odds**: outreach grounded in what each lead actually talks about.  
- **Reusable playbook**: the Markdown output can be used by any BD teammate.

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
| **Relevance** | 0–10 | Overlap between their topics and `key_topics` + ICP |
| **Influence** | 0–10 | Follower hints / role prominence (coarse brackets) |
| **Fit** | 0–10 | Voice/tone alignment with `campaign_tone` + goal |

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
  - Keep messages short (2–4 sentences).  
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

***

## 11) MVP Success Criteria

- With only a campaign brief, the skill:  
  - Finds at least 10–20 meaningful leads for a focused niche.  
  - Assigns reasonable A/B/C tiers with clear explanation.  
  - Produces Telegram‑ready DM drafts for all A/B leads.  
- All personalization is based on real, discoverable content; no hallucinated references.
