# The Closer

**The Closer** is a cold email writer and send bot for job seekers. It loads outreach targets from a JSON file, generates short personalized emails from a structured template, shows each message for human review, and sends (or simulates sending) through Gmail SMTP. Every attempt is logged to an audit CSV for proof and debugging.

The project is designed for safe, low-volume outreach: you preview every email, confirm each action (`send`, `draft`, or `skip`), and keep `DRY_RUN=true` until you are ready for a real test.

## How it works

1. **Load** contacts from `data/contacts.json`
2. **Generate** subject + body (company/role personalization, under 150 words)
3. **Preview** the full email in the terminal
4. **Confirm** — you choose send, draft, or skip
5. **Deliver** via dry-run simulation or Gmail SMTP
6. **Log** each outcome to `logs/outreach_log.csv`

## Project layout

| Folder | Purpose |
|--------|---------|
| `src/closer/cli/` | Orchestrator and guardrails |
| `src/closer/config/` | `.env` settings |
| `src/closer/domain/` | Models |
| `src/closer/input/` | JSON loader + validation |
| `src/closer/generation/` | Email templates |
| `src/closer/preview/` | Preview + confirmation |
| `src/closer/delivery/` | SMTP + dry-run |
| `src/closer/audit/` | CSV logging |
| `data/` | `contacts.json` |
| `logs/` | `outreach_log.csv` |
| `docs/` | Architecture, plan, submission checklist |

## Setup

Requires **Python 3.9+** (3.10+ recommended).

```bash
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

Optional tests:

```bash
pip install -r requirements-dev.txt
python -m pytest tests/ -q
```

## Configure `.env`

| Variable | Purpose |
|----------|---------|
| `SMTP_USER` | Your Gmail address |
| `SMTP_PASSWORD` | [Gmail App Password](https://myaccount.google.com/apppasswords) |
| `DRY_RUN` | `true` = no real sends (default) |
| `SEND_MODE` | `draft` or `send` (SMTP sends only on user `send`) |
| `MAX_OUTREACH_PER_RUN` | Volume cap (default `5`) |
| `INPUT_PATH` | Default `data/contacts.json` |
| `LOG_PATH` | Default `logs/outreach_log.csv` |

**Never commit `.env`** — only `.env.example` belongs in git.

## Run

### CLI

```bash
python main.py
```

### Streamlit UI (Stretch B)

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

Or:

```bash
PYTHONPATH=src streamlit run src/closer/ui/app.py
```

The UI reuses the same core modules (`generate_email`, `deliver_email`, `append_log`) and respects `DRY_RUN`.

Non-interactive checks:

```bash
python scripts/validate_mvp.py
python scripts/seed_sample_log.py   # writes logs/outreach_log.sample.csv
```

## Sending method

- **Provider:** Gmail via **SMTP** (`smtp.gmail.com`, port **587**, STARTTLS)
- **Library:** Python `smtplib`
- **Real send:** set `DRY_RUN=false`, choose **`send`** at the prompt
- **Drafts:** SMTP cannot create Gmail drafts; use `DRY_RUN=true` and choose `draft` to simulate

### Gmail App Password

1. Enable [2-Step Verification](https://myaccount.google.com/security)
2. Create an [App Password](https://myaccount.google.com/apppasswords)
3. Put the 16-character password in `SMTP_PASSWORD`

### Self-test

1. Set your email as the first contact in `data/contacts.json`
2. `DRY_RUN=false` in `.env`
3. Run `python main.py` → preview → type `send`
4. Verify Gmail **Sent** and a `sent` row in the log

## Safety

- Human review required for every email (FR3)
- `MAX_OUTREACH_PER_RUN` caps batch size (default 5)
- `DRY_RUN=true` by default
- Guardrails warn on missing `personalization_note` and sender/candidate mismatch
- No fabricated experience — templates only use fields you provide

## Submission

See [docs/SUBMISSION.md](docs/SUBMISSION.md) for the Phase 8 checklist. Place screenshots in `submission/screenshots/`.

Sample log for reviewers: `logs/outreach_log.sample.csv`

## Documentation

- [docs/problemStatement.md](docs/problemStatement.md)
- [docs/architecture.md](docs/architecture.md)
- [docs/implementation-plan.md](docs/implementation-plan.md)
