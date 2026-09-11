# DATABASE.md — SQLite Schema

## Tables

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);

CREATE TABLE transactions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    date TEXT NOT NULL,              -- ISO format YYYY-MM-DD
    merchant TEXT NOT NULL,
    amount REAL NOT NULL,            -- negative = debit, positive = credit
    category TEXT,                   -- e.g. Food, Rent, EMI, Subscription, Salary
    type TEXT,                       -- 'essential' | 'discretionary' | 'income'
    is_recurring INTEGER DEFAULT 0,  -- 0 or 1
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE goals (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    name TEXT NOT NULL,              -- e.g. "Emergency Fund"
    target_amount REAL NOT NULL,
    current_amount REAL DEFAULT 0,
    target_date TEXT NOT NULL,       -- ISO date
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE liabilities (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    name TEXT NOT NULL,              -- e.g. "Car Loan", "Credit Card"
    balance REAL NOT NULL,
    interest_rate REAL NOT NULL,     -- annual %, e.g. 14.5
    min_payment REAL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE alerts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    type TEXT NOT NULL,              -- 'overspend' | 'large_debit' | 'unused_sub'
    message TEXT NOT NULL,
    data_json TEXT,                  -- supporting numbers as JSON string
    created_at TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## Notes
- Keep `user_id` hardcoded to a single demo user if auth is skipped — just seed
  `users` with 2-3 rows for demo variety.
- `data_json` in alerts stores the "why" numbers shown in the UI's explainable
  recommendation cards.
- No migrations tooling needed — just a `init_db.py` script that runs `CREATE TABLE
  IF NOT EXISTS` on startup.
