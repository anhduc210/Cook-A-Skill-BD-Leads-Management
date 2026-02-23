
# BD Agent

**AI Business Development Agent for Lead Research, CRM & Outreach**

BD Agent helps you:

1. **Research** leads and enrich them with insights.  
2. **Manage** leads in a lightweight, insight‑driven CRM.  
3. **Reach out** automatically with personalized messages (Telegram in MVP, multi‑platform later).

> **MVP:** Telegram‑only outreach  
> **Future:** Multi‑platform (Telegram, X, LinkedIn, Email, …)

***

## Features

### 1. Research: Lead Discovery & Insights

- Create **research tasks** to define what kind of leads you want (e.g. “50 crypto KOLs talking about Polymarket”).  
- Enrich raw handles or candidate lists with AI‑generated insights:
  - Topics of interest (Polymarket, DeFi, NFTs, etc.)  
  - Communication style & personality (casual, memey, serious, shill, etc.)  
  - Short 1–2 sentence summary for each lead  
- Save selected candidates into the lead database with tags and notes.

> In MVP, research is **semi‑manual**: you provide candidate profiles/handles, the agent normalizes data and writes the insights for you.

***

### 2. Lead Management: Insight‑Driven Mini CRM

- Central **Lead Database** with:
  - Name, Telegram handle, source (research/manual)  
  - Status: `NEW`, `RESEARCHED`, `CONTACTED`, `REPLIED`, `QUALIFIED`, `CLOSED`  
  - Topics & interests (e.g. `["Polymarket", "Crypto", "Prediction Markets"]`)  
  - Personality notes (e.g. “loves memes, pro‑Polymarket”)  
  - Free‑form internal notes
- **Lead list view**:
  - Filter by status, topic, tag, source  
  - See last contacted / last replied timestamps
- **Lead detail view**:
  - Full profile  
  - AI insight summary  
  - Timeline of all Telegram messages (outbound + inbound)  
  - BD notes

> Future versions generalize this CRM to multiple channels (Telegram, X, LinkedIn, Email) while keeping a single unified profile per lead.

***

### 3. Outreach: Personalized Messaging Agent (Telegram MVP)

- Connect your **Telegram** account/bot.  
- Build **outreach campaigns** using segments from your lead database:
  - Example segments:
    - `status = RESEARCHED` and `topics includes "Polymarket"`  
    - `status = QUALIFIED` and `topics includes "Partnership"`  
- Define **campaign types**:
  - One‑off broadcasts (e.g. New Year greetings)  
  - Simple 2‑step sequences (initial message + follow‑up if no reply)
- Use **AI‑assisted templates**:
  - Variables: `{first_name}`, `{topics}`, `{personality}`, `{custom_note}`  
  - Tone adaptation based on personality (more casual/memey vs formal)
- **Safe sending**:
  - Daily send limit  
  - Time window (e.g. 09:00–21:00)  
  - Random delay between messages
- **Automatic updates**:
  - When a message is sent → lead status becomes `CONTACTED` (if new)  
  - When a reply is received → lead status becomes `REPLIED` and the reply is logged to the timeline

> Roadmap: extend the same campaign engine to X, LinkedIn & email with unified sequencing and channel preferences per lead.

***

## Tech Stack

- **Frontend:** React, TypeScript, Tailwind CSS  
- **Backend:** Node.js, Express  
- **Database:** PostgreSQL (or SQLite for quick setup)  
- **Messaging:** Telegram Bot API  
- **AI Layer:** Claude/OpenAI for insight generation & light personalization  
- **Job Queue:** BullMQ/Redis (for campaign scheduling & sending)

***

## Project Structure

```text
├── client/                         # Frontend (React + TypeScript + Tailwind)
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── features/
│       │   ├── research/           # Research tasks & lead enrichment UI
│       │   ├── leads/              # Lead list, detail, notes
│       │   └── campaigns/          # Telegram campaigns & stats
│       └── utils/
│
├── server/                         # Backend (Node.js + Express)
│   └── src/
│       ├── routes/
│       ├── controllers/
│       ├── services/
│       │   ├── researchAgent/      # AI insight generation
│       │   ├── leadService/        # Lead CRUD & pipeline logic
│       │   └── telegramOutreach/   # Campaign engine & sending
│       ├── integrations/           # Telegram, LLM, future X/LinkedIn
│       ├── jobs/                   # Background workers for campaigns
│       ├── models/                 # DB schemas (Lead, Campaign, Message, Task)
│       └── utils/
│
└── README.md
```

***

## Core Data Models (Conceptual)

```ts
// Lead
Lead {
  id: string;
  name?: string;
  telegramHandle: string;
  source: 'RESEARCH_TASK' | 'MANUAL';

  status: 'NEW' | 'RESEARCHED' | 'CONTACTED' | 'REPLIED' | 'QUALIFIED' | 'CLOSED';

  topics: string[];        // ["Polymarket", "DeFi", ...]
  personality?: string;    // "Casual, uses memes, pro-Polymarket"
  notes?: string;

  createdAt: Date;
  updatedAt: Date;
  lastContactedAt?: Date;
  lastRepliedAt?: Date;
}
```

```ts
// Campaign
Campaign {
  id: string;
  name: string;
  channel: 'TELEGRAM';     // Future: 'TELEGRAM' | 'X' | 'LINKEDIN' | ...
  type: 'BROADCAST' | 'SEQUENCE_2_STEP';

  segmentFilter: any;      // JSON: status, topics, tags, etc.
  step1Template: string;
  step2Template?: string;

  sendWindowStart: string; // "09:00"
  sendWindowEnd: string;   // "21:00"
  dailyLimit: number;

  status: 'DRAFT' | 'RUNNING' | 'PAUSED' | 'COMPLETED';
  createdAt: Date;
}
```

```ts
// Message
Message {
  id: string;
  leadId: string;
  campaignId?: string;
  channel: 'TELEGRAM';     // extensible to other channels
  direction: 'OUTBOUND' | 'INBOUND';
  content: string;
  sentAt: Date;
  telegramMessageId?: string;
}
```

```ts
// ResearchTask
ResearchTask {
  id: string;
  name: string;
  description: string;     // e.g. "50 crypto KOLs on Polymarket"
  status: 'RUNNING' | 'DONE';
  createdAt: Date;
}
```

***

## Quick Start (Conceptual)

1. **Set up the project**

```bash
# Install dependencies
npm install

# Run backend
cd server && npm run dev

# Run frontend
cd client && npm run dev
```

2. **Create research task**  
   - Open the app → “Research” → describe the type of leads you want → paste candidate handles → let the agent enrich & save.

3. **Segment your leads**  
   - Go to “Leads” → filter by topic (e.g. Polymarket) & status (`RESEARCHED`).

4. **Send a Telegram campaign**  
   - Go to “Campaigns” → create New Campaign → choose segment → write template → preview messages → start.  
   - Watch statuses move from `NEW`/`RESEARCHED` → `CONTACTED` → `REPLIED` as messages go out and replies come in.

***

## Roadmap (High Level)

- **Multi‑platform outreach**: Add X, LinkedIn, and email channels with shared sequences.  
- **Richer research**: Automatic discovery of leads from social/search APIs and public data sources.  
- **Smarter insights**: Deeper personality and preference modeling; auto‑suggest best channel and message style.  
- **Team view**: Multiple BD reps, shared pipelines, assignment, and collaboration.

***

You can paste this README as‑is into GitHub and then adjust naming/wording or strip sections you don’t need.
