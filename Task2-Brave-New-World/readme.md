# Dataset 2 — Brave New World

---

## Overview

This notebook runs the fraud detection pipeline on **Brave New World** — the second level of the Reply Mirror challenge. Compared to Dataset 1, it introduces more citizens, significantly more transactions, and richer communication data with heavier phishing campaigns targeting multiple users simultaneously.

The system identifies fraudulent transactions by combining pure Python signal extraction with a single LLM reasoning call per citizen, tracked end-to-end via Langfuse.

---

## Dataset

| Property | Value |
|---|---|
| Level | 2 — Brave New World |
| Citizens | 7 |
| Transactions | 522 |
| Extra data | SMS threads, email threads, GPS location pings |
| Audio files | None |

---

## What Changed from Dataset 1

- **7 citizens** instead of 3 — more users to investigate
- **522 transactions** instead of 80 — higher volume with more noise
- **Heavier phishing campaigns** — 77/77 phishing SMS events have transactions in the following 72h window, making correlation much more critical
- **More recipient diversity** — naive "new recipient" flagging no longer works; the preprocessor uses `unique_recipient` (appears only once in the entire dataset) instead
- **Multilingual user descriptions** — English, French, and German phishing vulnerability keywords in user profiles
- **3 users have IBAN mismatches** between `transactions.csv` and `users.json` — the preprocessor handles this with a sender_id abbreviation fallback

---

## Pipeline

```
For each citizen:
  preprocess_user()          ← pure Python, zero LLM calls
    ├── transaction signals   night activity, unique recipients,
    │                         rapid sequences, balance drain,
    │                         amount spike vs user median,
    │                         amount vs salary ratio
    ├── phishing signals      SMS + email scan for fake domains
    │                         (paypa1, amaz0n, ub3r, netfl1x)
    │                         and urgent keywords
    ├── phishing correlation  links phishing events to transactions
    │                         within 72h windows
    └── location anomalies   GPS ping vs in-person payment city
  run_fraud_scorer()         ← 1 LLM call via OpenRouter
    └── fraud_scorer_agent   create_agent with system prompt
write fraudulent_transactions.txt
send traces → Langfuse
```
