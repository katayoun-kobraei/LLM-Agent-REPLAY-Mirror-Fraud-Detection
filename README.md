# Reply Mirror — Fraud Detection Multi-Agent System

**Team:** Hashtag | **Challenge:** Reply AI Agent Challenge 2026

A multi-agent AI system that detects evolving financial fraud in the fictional 2087 digital city of Reply Mirror, built for the [Reply Mirror Challenge](https://challenges.reply.com).

---

## Overview

The system detects fraudulent transactions across three progressively complex datasets by combining pure Python signal extraction with a single LLM reasoning call per user. This design mainimizes token usage and latency while maintaining detection quality.

### Architecture

```
run_orchestrator()
  └── for each citizen:
        preprocess_user()        ← pure Python, zero LLM calls
          ├── Transaction signals (night, rapid sequence, balance drain, amount spike)
          ├── Phishing signals    (SMS + email keyword/domain scan)
          ├── Location anomalies  (GPS vs in-person payment mismatch)
          └── Audio signals       (Whisper transcription — DS3 only)
        run_fraud_scorer()       ← 1 LLM call via OpenRouter
          └── fraud_scorer_agent (create_agent with system prompt)
  └── write fraudulent_transactions.txt
  └── langfuse_client.flush()   ← send traces to Langfuse
```

### Key Design Decisions

- **1 LLM call per user** — all preprocessing is pure Python, keeping costs and latency minimal
- **`create_agent` pattern** — follows the official tutorial with `system_prompt` and LangChain
- **Langfuse tracking** — every run is traced with a unique `TEAM-ULID` session ID
- **Progressive complexity** — same pipeline handles all 3 datasets; DS3 adds Whisper audio

---

## Datasets

| Dataset | Level | Users | Transactions | Extra |
|---|---|---|---|---|
| The Truman Show | 1 | 3 | 80 | SMS, email, GPS |
| Brave New World | 2 | 7 | 522 | SMS, email, GPS |
| Deus Ex | 3 | 12 | 2017 | SMS, email, GPS, **48 MP3s** |


---



## Requirements

```
langchain==1.0.0
langchain-core
langchain-openai
langchain-community
langfuse
ulid-py
openai-whisper       # DS3 only
```

---

## Fraud Detection Logic

### Signals detected by the preprocessor (pure Python)

| Signal | Description |
|---|---|
| `night` | Transaction between 0am–4am |
| `unique_recipient` | Recipient appears only once in entire dataset |
| `rapid_sequence` | 3+ transactions within 1 hour from same sender |
| `balance_drain` | Balance drops below 5% of user's median balance |
| `amount_spike` | Amount is 4x+ the user's own median transaction amount |
| `ecommerce_no_desc` | E-commerce transaction with no description + unique recipient |
| `night_withdrawal` | Cash withdrawal between 0am–4am |
| `high_vs_salary` | Amount exceeds 2x monthly salary |
| Phishing SMS/email | Fake domains (amaz0n, paypa1, ub3r) or urgent keywords |
| Audio fraud signals | Whisper transcript contains 2+ fraud keywords (DS3) |
| Location mismatch | GPS city ≠ in-person payment city within 2h window |

### FraudScorerAgent rules

The LLM receives a compact structured evidence dict and applies these rules:

- Night + unique recipient → **flag**
- Balance drain → **flag**  
- Rapid sequence + any other signal → **flag**
- E-commerce with no description → **flag**
- Night withdrawal → **flag**
- Any transaction within 24h of high-severity phishing → **flag**
- Never flag: salary, rent, subscriptions, insurance, utility bills

---

## Observability

All runs are tracked in Langfuse at `https://challenges.reply.com/langfuse`.

Each run generates a unique session ID in the format `Hashtag-<ULID>` that groups all traces from one pipeline execution. The dashboard shows token usage, latency, and cost per user investigation.

---

## Team

| Name | GitHub |
|------|--------|
| Katayoon Kobraei | [@katayoon-kobraei](https://github.com/katayoon-kobraei) |
| Yashar Kalantari | [@YasharKalantari](https://github.com/YasharKalantari) |
| Farhad Norouzzadeh | [@Farhad-Norouzzadeh](https://github.com/Farhad-Norouzzadeh) |
| Sadegh Ofoghi | [@itsSadegh](https://github.com/itsSadegh) |

