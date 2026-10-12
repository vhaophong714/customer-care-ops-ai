# Customer Support SLA & CSAT Analysis

Analysis of 2,330 support tickets with **SQL Server and Power BI**: ticket volume, SLA,
customer satisfaction (CSAT) and agent productivity, summarised in a one-page dashboard.

## Table of Contents

1. [Business Problem](#1-business-problem)
2. [Tools](#2-tools)
3. [Dataset](#3-dataset)
4. [Project Structure](#4-project-structure)
5. [Data Cleaning](#5-data-cleaning)
6. [Dashboard](#6-dashboard)
7. [Key Findings](#7-key-findings)
8. [Recommendations](#8-recommendations)
9. [How to Run](#9-how-to-run)
10. [Limitations](#10-limitations)

---

## 1. Business Problem

A customer care team needs clear answers to four weekly questions:

1. **Volume** — how many tickets come in, through which channel, and when?
2. **SLA** — do we respond and resolve within the target time?
3. **CSAT** — what makes customers satisfied or not?
4. **Agents** — how is each agent doing on speed, completion and satisfaction?

## 2. Tools

- **SQL Server (T-SQL):** data checks, cleaning, KPI queries (CTEs, window functions)
- **Power BI (DAX):** one-page dashboard

## 3. Dataset

**[Technical Support Dataset](https://www.kaggle.com/datasets/suvroo/technical-support-dataset)** (Kaggle) —
2,330 tickets from 2023, 8 agents, 22 columns: created / first response / resolution times,
SLA deadlines, priority, channel (Email, Chat, Phone), topic, agent and tier,
CSAT survey score (1–5) and country. Column list: [`data/README.md`](data/README.md).

## 4. Project Structure

```
customer-support-sla-analysis/
├── README.md
├── data/
│   └── README.md                 # where to download the CSV + column list
├── sql/
│   ├── 01_data_cleaning.sql      # checks + tickets_clean table
│   └── 02_kpi_analysis.sql       # volume, SLA, CSAT, agent scorecard
├── powerbi/
│   ├── customer_support_sla.pbix
│   ├── measures.dax
│   ├── dashboard_guide.md
│   └── dashboard_mockup.png
└── images/
    └── dashboard.png
```

## 5. Data Cleaning

[`sql/01_data_cleaning.sql`](sql/01_data_cleaning.sql) checks the raw table, then builds one clean table, `tickets_clean`.

| Problem found | Tickets | What I did |
|---|---|---|
| Topic written two ways ("Pricing and licensing" / "Pricing and Licensing") | 68 | Merged into one |
| First response flagged "SLA Violated" although it came before the deadline (all 18–20 Feb 2023) | 14 | Recalculated both SLA flags from the timestamps |
| Unresolved ticket flagged "Within SLA" | 1 | Same |
| First response earlier than the ticket was created | 2 | Response time set to NULL |
| Agent interactions = 60 (all other values are 1–10) | 45 | Set to NULL |
| Closed before resolved | 42 | Close time not used |
| Survey score only exists for Closed tickets | 1,173 of 2,330 | CSAT measured on surveyed tickets only |

The SLA targets were not documented, so I worked them out from the deadline columns:

| Channel | First response | Resolution |
|---|---|---|
| Chat | 2 min | 24 h |
| Email | 60 min | 48 h |
| Phone | 2 min | 48 h |

Open tickets are measured on 4 Jan 2024, the last event in the file.

## 6. Dashboard

![Dashboard](images/dashboard.png)

One page with 6 KPIs, slicers for channel, priority and agent group, and 6 visuals.
Build steps: [`powerbi/dashboard_guide.md`](powerbi/dashboard_guide.md) · Measures: [`powerbi/measures.dax`](powerbi/measures.dax)

| Tickets | First response SLA | Resolution SLA | Open tickets | CSAT | Median resolution |
|---|---|---|---|---|---|
| 2,330 | 87.3% | 66.4% | 418 | 56.7% | 21.9 h |

CSAT = share of survey scores 4–5. Average score 3.51 / 5 from 1,173 surveys.

## 7. Key Findings

### 7.1 Volume: steady all year, half of it about two topics

- 159–224 tickets a month with no clear peak.
- About 7–8 tickets a day on weekdays and 3–4 at weekends, spread across all 24 hours.
- Email 53%, Chat 36%, Phone 11%.
- **Product setup (27%)** and **Pricing and licensing (23%)** make up half of all tickets.

### 7.2 SLA: the real problem is tickets that never get closed

- First response SLA is met for 87.3% of tickets, but **Phone is lowest at 76.0%**
  (59 of 246 calls missed the 2-minute target), compared with 91.7% for Email.
- Resolution SLA is only 66.4%. Of the **784 breaches, 418 are tickets that were never resolved**.
  Counting only resolved tickets, 80.9% met the SLA.
- The 418 open tickets were created in every month of the year, and
  **242 of them had been open for more than 6 months**.
- No single agent causes this: each agent has 12–21% of their tickets still open (chi-square p = 0.20).
  It looks like a missing follow-up process, not one person's problem.

### 7.3 CSAT: speed did not change satisfaction, topic and agent did

- Tickets resolved late scored the same as tickets on time: **57.3% vs 56.5%** satisfied (p = 0.89).
- **Product setup has the lowest CSAT: 51.3%** vs 58.7% for all other topics (p = 0.03).
- **One agent stands out:** Connor Danielovitch reaches 69.0% CSAT vs 54.6% for the rest (p < 0.001).

### 7.4 Agents: quality and completion must be read together

- Tier 1 agents handle about 354 tickets each in the year, Tier 2 about 187.
- The agent with the best CSAT (Connor, 69.0%) also has the highest share of open tickets (21.3%).
  The agent with the best resolution SLA (Heather Urry, 76.3%, only 12.4% open) has one of the lowest CSAT scores (52.2%).
- A scorecard that shows only one metric would reward the wrong behaviour.

## 8. Recommendations

| # | Finding | Action | How to measure |
|---|---|---|---|
| 1 | 418 tickets never closed, many for months | Send the team lead a daily list of overdue tickets and clean up the backlog once | Open tickets older than 30 days → 0 |
| 2 | Phone misses the 2-min target 24% of the time | Check staffing at busy hours; offer a call-back option | Phone first response SLA ≥ 85% |
| 3 | Product setup: biggest topic, lowest CSAT | Write step-by-step setup guides and use them as chatbot / FAQ answers | Product setup CSAT and ticket share |
| 4 | Connor's CSAT is much higher | Review some of his tickets and share what works; pair it with closing tickets on time | CSAT gap between agents |
| 5 | Speed alone does not drive CSAT | Add one survey question: "Was your problem solved?" | Share of "yes" by topic |

Recommendation 1 is built as an automation in my follow-up project,
**[AI Ticket Triage with n8n](https://github.com/vhaophong714/AI-Ticket-Triage-n8n)**.

## 9. How to Run

1. Create a database `Customer_Care` in SQL Server.
2. Download the CSV (see [`data/README.md`](data/README.md)) and import it with
   SSMS → Tasks → **Import Flat File**, table name `support_tickets_raw`.
3. Run `sql/01_data_cleaning.sql`, then `sql/02_kpi_analysis.sql`.
   Expected results are written as comments above each query.
4. Build the dashboard with [`powerbi/dashboard_guide.md`](powerbi/dashboard_guide.md).

## 10. Limitations

- The ticket data is a public sample, not a real company's data. Some patterns
  (for example, the same volume at 3 a.m. as at 3 p.m.) may not match real life.
- The data does not say why tickets stay open, so I cannot tell whether the work is unfinished
  or the status was simply not updated. Both need the same fix: a regular follow-up.
