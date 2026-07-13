# NAM LinkedIn ABM Dashboard — H1 2026

Streamlit dashboard measuring LinkedIn's incremental contribution to pipeline across 9 NAM ABM campaigns. Deployed on BigBrain (auto-deploys on push to `main` on `shaimas-dot/slg-leads-dashboard`).

**Owner:** shaimas@monday.com  
**Last updated:** 2026-07-13

---

## Critical deployment constraints

- **No scipy** — not available on BigBrain's Streamlit environment
- **No file reads at runtime** — no `open()`, `pd.read_csv()`, etc.
- **All data must be hardcoded** in `app.py` as Python dicts/lists
- To update: query Snowflake (via Kremer MCP), compute aggregates locally, paste into `app.py`, push to GitHub

---

## Dashboard tabs

| Tab | Routing condition | What it covers |
|-----|-----------------|----------------|
| 🔵 NAM Marketing (Work Mgmt) | `"NAM Marketing" in page` | Deep-dive: Marketing ANA campaign funnel, engagement, creative performance |
| 🟣 NAM PMO | `"PMO" in page` | Deep-dive: PMO campaign funnel, spend, ROI |
| 🟢 ANA ABM — Account Report | `"ANA ABM" in page` | Account-level report for ANA cohort |
| 🛡️ HUB International — 1:1 ABM | `"HUB International" in page` | Single-account 1:1 report (account ID 19755708) |
| 📈 Incremental Growth | `"Incremental Growth" in page` | Unified cross-campaign analysis — see structure below |

**Important:** All page routing uses positive `if "X" in page` conditions. Never use negative conditions (`not in`) — they cause multiple tabs to render simultaneously.

---

## Incremental Growth tab structure

1. **H2 Recommendations** — Scale/Maintain/Optimize/Pause table (top of page)
2. **Incrementality Framework** — 3 attribution proxies + holdout caveat
3. **Act 1 — Reach** — Impression tier distribution, engagement rate per tier, tier × funnel correlation
4. **Act 2 — Leads** — 31K leads by FIRST_LEAD_SOURCE × campaign, lead density
5. **Act 3 — Event Influence** — Event lead STATUS breakdown, event vs non-event qual rate comparison
6. **Act 4 — Pipeline** — Aware%/Opp%/Cust% per campaign
7. **Act 5 — Efficiency** — Cost/opp per campaign, full summary table
8. **Act 6 — H2 Simulator** — Budget → projected opps → customers → ARR

---

## The 9 SFDC campaigns

Full name prefix: `Other_Account_Based_Marketing_NAM_Q12026_`

| Campaign | SFDC ID | Suffix | TAL |
|----------|---------|--------|-----|
| Marketing ANA | `701av00000Q3zkmAAB` | `US_Land_Marketing_ANA_H1` | 1,269 |
| PMO | `701av00000Q3zkoAAB` | `US_Land_PMO_H1` | 1,404 |
| CRO | `701av00000Q3zkpAAB` | `US_Land_CRO_H1` | 1,440 |
| Retail | `701av00000Q3zkqAAB` | `US_Land_Retail_H1` | 530 |
| MKTG Whitespace | `701av00000Q3zkrAAB` | `US_Land_MKTG_Whitespace_H1` | 636 |
| SLED Counties | `701av00000Q3zksAAB` | `US_Land_SLED_Counties_H1` | 2,402 |
| SLED Higher Ed | `701av00000Q3zktAAB` | `US_Land_SLED_Higher_Ed_H1` | 658 |
| SLED Existing | `701av00000Q3zkuAAB` | `US_Land_SLED_Existing_H1` | 330 |
| SLED Counties WS | `701av00000Q3zkvAAB` | `US_Land_SLED_Counties_Whitespace_H1` | 1,068 |

---

## Key Snowflake tables

| Table | Key columns | Used for |
|-------|-------------|----------|
| `MARKETING.L3.V_ABM_COMPANIES_FUNNEL` | COMPANY_NAME, LIFECYCLE_STAGE (0–9), STAGE_END_DATE | Funnel stage per account. Filter `STAGE_END_DATE IS NULL` for current. Stages: Targeted(0) → Aware(1) → Engage(2) → MQA(3) → Opportunity(4) → Customer(5) → Expansion(6-9) |
| `BIGBRAIN.L2.RAW_SALESFORCE_LEADS` | ABM_CAMPAIGN_LEAD, FIRST_LEAD_SOURCE, LEAD_SOURCE, STATUS | Lead source breakdown and event lead statuses |
| `BIGBRAIN.L2.RAW_SALESFORCE_CAMPAIGN_MEMBERS` | CAMPAIGN_ID, CAMPAIGN_NAME, STATUS, TYPE, LEAD_ID, CONTACT_ID, ACCOUNT_ID | TAL account lists per campaign |
| `MARKETING.STG.STG_ABM_LEADS_SIGNUPS` | LEAD_ID, ABM_CAMPAIGN_NAME, LEAD_STATUS, SIGNUP_CREATED_AT | Date-filtered leads (use for H1-specific counts) |
| `MARKETING.L3.FACT_ABM_ENGAGEMENT_METRICS` | SIGNUPS_ABM (temporal, not attributed), web_cs, events | Engagement signals. Note: SIGNUPS_ABM is anyone at a TAL company who signed up after campaign start — NOT LinkedIn-attributed |
| `MARKETING.L3.FACT_LINKEDIN_CAMPAIGNS_DAILY` | spend (CRQ/ADN) | LinkedIn spend per campaign |
| `MARKETING.STG.STG_ABM_TARGETS` | COMPANY_NAME, COMPANY_ID, DOMAIN | Bridge table for account matching |

---

## Lead source classification (FIRST_LEAD_SOURCE)

| Dashboard label | Raw SFDC values |
|----------------|-----------------|
| Tools (Outbound) | `Tools` |
| Signup | `Big Brain - Signup` |
| Contact Sales | `Website - Contact Sales`, `Landing Page - Contact Sales`, `Mobile - Contact Sales` |
| Event | `Event`, `Online Event` |
| Other | Content, Platform, Partner, Vendors, Webinar, etc. |

**Pattern:** SLED campaigns are 85–97% Signup (gov/edu orgs). Commercial campaigns (ANA, PMO, Retail, CRO) are 52–65% Tools (Outbound).

---

## LinkedIn CSV (NOT in Snowflake)

File: `monday ABM - Company Journey Tool @ Jan - Jun 2026 - NAM Land Ad Sets (1 Jan - 13 Jul).csv`  
104,390 rows (weekly per company). Key columns: `company_name`, `total_impressions`, `total_engagements`, `total_clicks`.  
Many cells show "Below Reporting Threshold" — skip these when aggregating.

**To re-aggregate:** group by `company_name`, sum numeric fields, skip "Below Reporting Threshold" strings. Bucket by total impressions per company into tiers.

**Tier distribution (Jan 1–Jul 13 2026):**
- None (0): 2,499 accounts, 0.00% eng rate
- Minimal (1–99): 4,302 accounts, 0.98% eng rate
- Low (100–999): 3,318 accounts, 1.08% eng rate
- Medium (1k–5k): 1,123 accounts, 1.27% eng rate
- Med-High (5k–10k): 176 accounts, 1.45% eng rate
- High (>10k): 91 accounts, 1.59% eng rate

---

## Key findings — H1 2026

- **TAL coverage:** 95.7% of TAL (9,382/9,813 accounts) reached on LinkedIn
- **Total person-level leads:** 31,173 (FIRST_LEAD_SOURCE, all-time, RAW_SALESFORCE_LEADS)
- **Event leads:** 251 total — **10 Qualified** (PMO: 6, SLED Existing: 2, Retail: 2)
- **Event qual rate:** 4.0% vs 0% for all other sources — events produce 100% of Qualified leads
- **Contact Sales leads:** 530 high-intent (PMO leads with 168)
- **Lead status:** As of Jul 2026, 0 MQL/SQL across all 31K leads — enterprise cycle is 6–18 months
- **Most efficient campaign:** SLED Existing ($367/opp, 30.6% opp rate) — underinvested at $37K H1
- **Least efficient:** Marketing ANA ($8,037/opp) — highest spend, rebalance toward events

## H2 recommendations

| Campaign | Action | Rationale |
|----------|--------|-----------|
| PMO | 🟢 Scale | 35.2% opp rate, $1,151/opp, 6 event Qualified leads |
| SLED Existing | 🟢 Scale | 30.6% opp rate, $367/opp, underinvested at $37K |
| CRO | 🟢 Scale | 19.4% opp rate, $177/opp, low spend |
| Retail | 🟡 Maintain | 27.9% opp rate, event leads qualifying |
| SLED Higher Ed | 🟡 Maintain | Near-zero spend, monitor before scaling |
| Marketing ANA | 🟡 Optimize | $8,037/opp — rebalance toward events |
| MKTG Whitespace | 🔴 Reassess | 6.0% opp rate, question TAL quality |
| SLED Counties | 🔴 Reassess | 3.2% opp rate, large TAL not converting |
| SLED Counties WS | 🔴 Pause | 0.8% opp rate, near-zero pipeline |

---

## Incrementality proxies (no holdout exists)

1. **Impression frequency lift:** High-tier accounts (>10K imps) engage at 1.59% vs 0.98% minimal = +62% lift
2. **Contact Sales signal:** 530 TAL accounts requested demo after seeing LinkedIn
3. **Event qualification rate:** 4.0% vs 0% — events (part of ABM motion) produce only Qualified leads

**Recommended for H2:** Exclude 15–20% of one campaign's TAL from LinkedIn in Q3. Compare Q4 funnel progression as the holdout control. This becomes the 2027 budget case.

---

## Kremer MCP (Snowflake queries)

- Tool: `mcp__Kremer__data-expert-agent` + `mcp__Kremer__check-query-status`
- Very slow (2–5+ min per query). Poll with `check-query-status` after 30–60s.
- Large result sets (>10K rows) sometimes return empty — split into smaller per-campaign queries
- Sessions expire after ~10 minutes of inactivity — start a new `data-expert-agent` call with a fresh question
- Reuse `sessionId` to chain follow-up queries in the same session
