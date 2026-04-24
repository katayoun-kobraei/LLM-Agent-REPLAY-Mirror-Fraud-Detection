# Dataset 3 — Deus Ex

---

## Overview

This notebook runs the fraud detection pipeline on **Deus Ex** — the third and most complex level of the Reply Mirror challenge. It introduces 12 citizens, over 2000 transactions, and a new data source: **48 MP3 audio files** containing phone call recordings that must be transcribed and analyzed for fraud signals.

The system identifies fraudulent transactions by combining Whisper audio transcription, pure Python signal extraction, and a single LLM reasoning call per citizen, tracked end-to-end via Langfuse.

---

## Dataset

| Property | Value |
|---|---|
| Level | 3 — Deus Ex |
| Citizens | 12 |
| Transactions | 2,017 |
| Extra data | SMS threads, email threads, GPS location pings |
| Audio files | **48 MP3 phone call recordings** |

---

## What Changed from Dataset 2

- **12 citizens** instead of 7 — largest user set so far
- **2,017 transactions** instead of 522 — high volume, ~160 transactions per citizen
- **48 MP3 audio files** — phone call recordings per citizen that must be transcribed using Whisper and scanned for fraud keywords
- **High recipient diversity** — up to 39% of each citizen's transactions go to unique recipients, making simple `new_recipient` signals too noisy; `unique_recipient` (global dataset uniqueness) is used instead
- **Multilingual audio** — recordings may be in English, German, French, or Italian; Whisper handles all automatically with `language=None`
- **GPU incompatibility** — Kaggle's P100 GPU (CUDA 6.0) is incompatible with the installed PyTorch version; Whisper runs on CPU with `CUDA_VISIBLE_DEVICES=""` to force CPU mode

---

## Pipeline

```
transcribe_audio_files()     ← Whisper tiny on CPU (~3-5 min for 48 files)
  └── {first_name: [{file, datetime, text}]}

For each citizen:
  preprocess_user()          ← pure Python, zero LLM calls
    ├── transaction signals   night activity, unique recipients,
    │                         rapid sequences, balance drain,
    │                         amount spike vs user median,
    │                         amount vs salary ratio
    ├── phishing signals      SMS + email scan for fake domains
    │                         and urgent keywords
    ├── audio signals         Whisper transcript keyword scan
    │                         (2+ fraud keywords → signal)
    ├── phishing correlation  links all attack events to transactions
    │                         within 72h windows
    └── location anomalies   GPS ping vs in-person payment city
  run_fraud_scorer()         ← 1 LLM call via OpenRouter
    └── fraud_scorer_agent   create_agent with system prompt
write fraudulent_transactions.txt
send traces → Langfuse
```
