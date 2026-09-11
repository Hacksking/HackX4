# FRONTEND.md — Build Spec (React + Vite + Tailwind)

## Folder Structure
```
frontend/
  src/
    api/
      client.js          # fetch wrapper, base URL config
    mock/
      transactions.json
      dashboard.json
      cashflow.json
      goals.json
      liabilities.json
      alerts.json
    pages/
      UploadPage.jsx
      DashboardPage.jsx
      GoalsPage.jsx
      LiabilitiesPage.jsx
      SimulationPage.jsx
    components/
      CategoryChart.jsx
      CashflowChart.jsx
      RecommendationCard.jsx   # shows suggestion + expandable "why" numbers
      AlertBanner.jsx
      GoalForm.jsx
      LiabilityRankCard.jsx
    App.jsx                    # routing
    main.jsx
```

## Pages & What They Need

### 1. UploadPage
- File input (CSV) → POST `/upload`
- On success, navigate to Dashboard

### 2. DashboardPage
- Fetch `/dashboard/summary` + `/cashflow`
- Category spend chart (bar or pie) — `CategoryChart`
- Cash-flow projection line chart — `CashflowChart`
- List of detected recurring subscriptions
- Alerts banner at top — fetch `/alerts`

### 3. GoalsPage
- `GoalForm` — name, target amount, target date → POST `/goals`
- List existing goals with `monthly_contribution_required` shown via
  `RecommendationCard`

### 4. LiabilitiesPage
- Fetch `/liabilities`
- Render as ranked list, each a `LiabilityRankCard` showing balance, rate,
  true annual cost, and rank reason (expandable "why")

### 5. SimulationPage (bonus, build last)
- Simple form: scenario type dropdown + amount input → POST `/simulate`
- Show before/after side-by-side (reuse `CashflowChart` twice or overlay)
- Show `narration` text from response

## RecommendationCard Component (reused everywhere)
Every recommendation, alert, or ranking must use this pattern:
```
[ Headline recommendation ]
[ Small "Why?" toggle ]
  -> expands to show the raw numbers used to compute it
```
This directly satisfies the "explainable recommendations" requirement — build
this component first, reuse it on Dashboard, Goals, Liabilities, Alerts.

## Build Order
1. Scaffold routes in `App.jsx`, set up Tailwind
2. Build `mock/*.json` matching `API_CONTRACT.md` exactly
3. Build `RecommendationCard` (reused everywhere — build once, early)
4. Build UploadPage + DashboardPage against mock data
5. Build GoalsPage + LiabilitiesPage against mock data
6. Switch `api/client.js` base URL from mock files to real backend once ready
7. Build SimulationPage last, only if time remains

## Notes
- Use `api/client.js` as the single place that decides mock vs real — flip one
  flag/env var to switch, so no component code needs to change.
- Keep charts responsive but don't over-engineer styling until Part 3 (polish pass).
