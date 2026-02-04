# 💬 Telegram Message Exporter (macOS Desktop)

[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Platform](https://img.shields.io/badge/platform-macOS-111111.svg)](#)
[![Telegram](https://img.shields.io/badge/Telegram-Desktop-2CA5E0.svg)](https://desktop.telegram.org/)
[![Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://black.readthedocs.io/)
[![Ruff](https://img.shields.io/badge/lint-ruff-262626.svg)](https://docs.astral.sh/ruff/)
[![Pylint](https://img.shields.io/badge/lint-pylint-ffcd00.svg)](https://pylint.readthedocs.io/)

A professional, offline recovery and export tool for **Telegram Desktop (macOS)**. It decrypts the local `db_sqlite` using `.tempkeyEncrypted` and produces a clean, readable transcript in **HTML**, **Markdown**, or **CSV**.

## Table of Contents

- [Overview](#overview)
- [Motivation & Use Case](#motivation--use-case)
- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Export Formats](#export-formats)
- [Example Output](#example-output)
- [Date Filtering](#date-filtering)
- [Safety & Privacy](#safety--privacy)
- [Versioning](#versioning)
- [Quality Checks](#quality-checks)
- [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [Credits](#credits)

---

## Overview

Telegram Desktop stores messages locally in an encrypted SQLite database. This tool:

1. Decrypts `db_sqlite` using `.tempkeyEncrypted`
2. Parses the Postbox key/value format
3. Exports a clean transcript that a non‑technical user can read

It is designed for **offline recovery** on a Mac where the local cache still exists.

---

## Motivation & Use Case

Telegram does not provide server‑side recovery for deleted chats. In real‑world scenarios (accidental deletion, account changes, device loss, or audit requirements), the **only remaining source of truth** can be the local encrypted cache on a macOS device.

This project was created after a family conversation was removed with no way to restore it via Telegram’s servers. The local Mac still had the encrypted cache, so this tool was built to **recover what remained locally** and convert it into a clean, shareable export.

If you need a **defensible, offline transcript** from Telegram Desktop’s local database, this provides a reliable and repeatable path.

---

## Key Features

- **Offline decryption** using Telegram’s local key format (dbKey + dbSalt)
- **Human‑readable exports** with names, timestamps, and link handling
- **Modern HTML transcript** with date jump + back‑to‑top button
- **CSV export** for analysis or spreadsheets
- **Date filters** for targeted ranges
- **Best‑effort peer mapping** for clean names

---

## Prerequisites

- **macOS** with Telegram Desktop data present
- **Python 3.10+** (tested on 3.11–3.13)
- **Virtual environment recommended**

---

## Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

If you prefer a requirements file:

```bash
pip install -r requirements.txt
```

---

## Quick Start

### 1. Decrypt the database

```bash
telegram-exporter decrypt \
  --key ~/Library/Group\ Containers/6N38VWS5BX.ru.keepcoder.Telegram/stable/.tempkeyEncrypted \
  --db  ~/Library/Group\ Containers/6N38VWS5BX.ru.keepcoder.Telegram/stable/account-*/postbox/db/db_sqlite \
  --out plaintext.db
```

If **Passcode Lock** is enabled in Telegram Desktop:

```bash
TG_LOCAL_PASSCODE="your-passcode" \
  telegram-exporter decrypt --key <key> --db <db> --out plaintext.db
```

### 2. Find the peer ID

```bash
telegram-exporter list-peers --db plaintext.db --search "Alice"
```

### 3. Export a readable transcript

```bash
telegram-exporter export \
  --db plaintext.db \
  --peer-id 123456789 \
  --me-name "Me" \
  --format html \
  --out chat_export.html
```

---

## Export Formats

### HTML (recommended)
Clean, modern transcript with date jump and back‑to‑top.

```bash
telegram-exporter export --db plaintext.db --peer-id 123456789 --format html --me-name "Me" --out chat_export.html
```

### Markdown
Readable, portable, easy to email.

```bash
telegram-exporter export --db plaintext.db --peer-id 123456789 --format md --me-name "Me" --out chat_export.md
```

### CSV
For spreadsheets or analysis.

```bash
telegram-exporter export --db plaintext.db --peer-id 123456789 --format csv --out chat_export.csv
```

---

## Example Output

### HTML (snippet)

```html
<header class="glass header-panel">
  <div class="brand">
    <div class="logo">💬</div>
    <div class="title-area">
      <h1>Alice Example</h1>
      <p class="subtitle">Recovery export for Telegram Desktop (macOS)</p>
    </div>
  </div>
  <div class="badge glass"><span class="dot"></span><span class="text">Ready</span></div>
</header>
```

### Markdown (snippet)

```markdown
# Telegram Chat History: Alice Example

**Exported:** 2026-02-04 16:05:12
**Total Messages:** 418

## Wednesday, February 04, 2026

**14:13:09 — Me**

3h48 is good also
```

### CSV (snippet)

```csv
date,time,timestamp,direction,speaker,text,peer_id,author_id
2026-02-04,14:13:09,1770214389,out,Me,"3h48 is good also",23556879,23556879
```

---

## Date Filtering

Export only a range (inclusive):

```bash
telegram-exporter export \
  --db plaintext.db \
  --peer-id 123456789 \
  --start-date 2024-01-01 \
  --end-date 2024-12-31 \
  --format html \
  --out chat_2024.html
```

Formats supported:
- `YYYY-MM-DD`
- `YYYY-MM-DDTHH:MM:SS`
- Unix timestamp (seconds)

---

## Safety & Privacy

- Keep the Mac **offline** during recovery to avoid sync deletions.
- Media files (if cached) live in:
  `~/Library/Group Containers/6N38VWS5BX.ru.keepcoder.Telegram/stable/account-*/files/`
- If decryption fails, retry with `--debug` to see which SQLCipher profile succeeds.

---

## Versioning

The canonical version is stored in `VERSION` and exposed via:

```bash
telegram-exporter --version
```

To bump the version:

```bash
# patch (0.1.1 -> 0.1.2)
./scripts/bump_version.py patch

# minor (0.1.1 -> 0.2.0)
./scripts/bump_version.py minor

# major (0.1.1 -> 1.0.0)
./scripts/bump_version.py major

# explicit
./scripts/bump_version.py --set 1.2.3
```

---

## Quality Checks

Run formatting and linting locally:

```bash
black src/telegram_message_exporter telegram_exporter.py
ruff check src/telegram_message_exporter telegram_exporter.py
pylint src/telegram_message_exporter telegram_exporter.py
```

---

## Troubleshooting

**“file is not a database”**
- Key and DB are mismatched (wrong snapshot or wrong account path)
- Passcode Lock is enabled but not provided

**Passcode Lock on**
- Use `TG_LOCAL_PASSCODE` or `--passcode`

**mmh3 errors**
```bash
pip install mmh3==4.1.0
```

---

## Project Structure

```
telegram-message-exporter/
├── pyproject.toml                     # Packaging metadata + CLI entrypoint
├── telegram_exporter.py               # Convenience wrapper (no install)
├── src/
│   └── telegram_message_exporter/
│       ├── __init__.py                # Package entrypoint
│       ├── __main__.py                # python -m entrypoint
│       ├── cli.py                     # Argument parsing + commands
│       ├── crypto.py                  # SQLCipher + tempkey handling
│       ├── db.py                      # DB heuristics + message extraction
│       ├── exporters.py               # HTML / Markdown / CSV
│       ├── postbox.py                 # Postbox parsing utilities
│       ├── models.py                  # Message data model
│       ├── utils.py                   # Date + link helpers
│       └── hashing.py                 # Murmur3 helper
├── requirements.txt                   # Python dependencies
└── README.md
```

---

## Credits

This project builds on community reverse‑engineering work. The initial breakthrough and reference implementation for Telegram Desktop’s local key format and Postbox structure comes from **@stek29** (GitHub). This tool extends those ideas into a polished, end‑user‑friendly CLI and export workflow.

- https://gist.github.com/stek29/8a7ac0e673818917525ec4031d77a713

---

For enhancements or alternate export styles, feel free to open a PR (or fork and submit one). We’ll review and merge solid improvements—this repo is meant to be a good base to build on.
