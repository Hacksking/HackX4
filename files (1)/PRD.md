# PRD — Personal Financial Health Assistant (FinTech PS #3)

## 1. Problem
Users' financial life is scattered across salary, card statements, EMIs, insurance,
and small investments — no single view of where money goes, and no next-step advice.

## 2. Solution
An AI-powered assistant that ingests a user's transactions, classifies spending,
projects cash flow, tracks goals, ranks liabilities, and gives explainable,
numbers-backed recommendations.

## 3. Target Demo User
A salaried individual with: monthly salary credit, rent, 1-2 EMIs, 3-4 recurring
subscriptions, discretionary spends, 1 savings goal, 1-2 active loans/cards.

## 4. Core Features (MVP — must ship)
1. **Upload & Ingest** — CSV upload of transactions
2. **Expense Intelligence** — categorize (rules + LLM fallback), flag recurring subs,
   essential vs discretionary
3. **Cash Flow View** — project month-end balance from committed in/outflows
4. **Goal-Based Planning** — user sets a goal, system computes required monthly saving
5. **Liability Intelligence** — rank loans/cards by true interest cost
6. **Actionable Alerts** — overspend, upcoming large debit, unused subscription
7. **Explainable Recommendations** — every suggestion shows the numbers behind it

## 5. Bonus Feature (only if time allows)
**Simulation Mode** — user inputs a life change (new EMI, rent increase, income
change) → system reruns cash flow + goals → shows before/after impact.

## 6. Non-Goals (explicitly out of scope for hackathon)
- Real bank account linking / Account Aggregator integration (mention as future work)
- User authentication / multi-user security
- Mobile app
- Real-time transaction sync

## 7. Tech Stack
- Frontend: React + Vite + Tailwind + Recharts
- Backend: FastAPI (Python)
- Database: SQLite
- AI: LLM API (Claude/GPT) for categorization fallback + recommendation narration
  — **never for math**, only for classification and explanation text

## 8. Design Principle
> Deterministic code calculates the numbers. AI classifies ambiguous data and
> explains the numbers in plain language. AI never invents a financial figure.

## 9. Success Criteria (demo-ready checklist)
- [ ] Upload CSV → transactions appear categorized on dashboard
- [ ] Recurring subscriptions auto-detected and listed
- [ ] Cash flow chart shows projected month-end balance
- [ ] User can create a goal and see required monthly contribution
- [ ] Liabilities ranked with "pay this first" recommendation + reasoning
- [ ] At least 2 alerts trigger correctly on demo data
- [ ] Every recommendation card shows an expandable "why" with real numbers
- [ ] (Bonus) Simulation mode shows before/after on one scenario

## 10. Related Docs
- `DATABASE.md` — schema
- `API_CONTRACT.md` — endpoint specs
- `BACKEND.md` — backend build spec
- `FRONTEND.md` — frontend build spec
- `AI_INTEGRATION.md` — LLM prompts and integration rules
