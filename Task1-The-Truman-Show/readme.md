# Dataset 1 — The Truman Show

---

## Overview

This notebook runs the fraud detection pipeline on **The Truman Show** dataset — the first and smallest level of the Reply Mirror challenge. It contains 80 transactions across 3 citizens in a fictional 2087 digital city where MirrorPay processes all financial activity.

The system identifies fraudulent transactions by combining pure Python signal extraction with a single LLM reasoning call per citizen, tracked end-to-end via Langfuse.

---

## Dataset

| Property | Value |
|---|---|
| Level | 1 — The Truman Show |
| Citizens | 3 (Karl-Hermann, Tracy, Alain) |
| Transactions | 80 |
| Extra data | SMS threads, email threads, GPS location pings |
| Audio files | None |

---

## Pipeline

```
For each citizen:
  preprocess_user()          ← pure Python, zero LLM calls
    ├── transaction signals   night activity, new recipients,
    │                         rapid sequences, balance drain,
    │                         amount vs salary ratio
    ├── phishing signals      SMS + email scan for fake domains
    │                         (paypa1, amaz0n, ub3r) and
    │                         urgent keywords
    └── location anomalies   GPS ping vs in-person payment city
  run_fraud_scorer()         ← 1 LLM call via OpenRouter
    └── fraud_scorer_agent   create_agent with system prompt
write fraudulent_transactions.txt
send traces → Langfuse
```
