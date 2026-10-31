# Jarvis Reports

A single-file reporting dashboard for voice assistants and automation scripts. It records every command and task, tracks system health, exports reports, and includes a multilingual AI chat. It needs only the Python standard library.

## Features

- **Activity log**: record commands with their status (success, failed, error) and get a suggested fix for each error.
- **Task tracking**: create or update tasks by ID (pending, running, completed, failed) without duplicates.
- **System monitor**: CPU, RAM and disk usage with history charts. A snapshot is saved every minute in the background.
- **Reports**: generate full, activity, task or system reports, save them, and export as TXT, CSV or PDF.
- **AI chat**: streaming chat powered by the Anthropic API. Jarvis replies in the language you write in (English, Urdu, Hindi, Roman Urdu).
- **Multilingual dashboard**: English, اردو and हिन्दी, with right-to-left support.
- **Voice integration**: `voice_command("show report")` opens the dashboard from recognized speech.
- **Security**: local-only by default, optional password (PBKDF2), login lockout, Host and Origin checks, CSP headers, CSV formula-injection protection, chat rate limiting.
- **Built-in tests**: 24 self-tests, no extra tools required.

## Requirements

- Python 3.8+
- Optional: `psutil` for more accurate CPU and RAM readings
- Optional: an Anthropic API key for the AI Chat tab

## Quick start

```bash
python jarvis_reports.py            # dashboard at http://127.0.0.1:8765
python jarvis_reports.py --demo     # start with sample data
python jarvis_reports.py --test     # run the self-tests
```

## AI chat

```bash
export ANTHROPIC_API_KEY="your-key"           # Windows: set ANTHROPIC_API_KEY=your-key
export JARVIS_CHAT_MODEL="claude-sonnet-5-5"  # optional
python jarvis_reports.py
```

Chat messages are never stored in the activity log.

## Use inside your assistant

```python
from jarvis_reports import log_activity, upsert_task, track, start, voice_command

log_activity("open chrome", "launched browser", "success")
log_activity("weather", "api call", "error", "ConnectionError: ...")

upsert_task("t1", "Send email", "pending")   # later: "completed" or "failed"

@track("play music")                          # logs success or error automatically
def play_music(): ...

start(background=True)                        # run the dashboard inside your app

reply = voice_command(recognized_text)        # returns None if it is not a report command
```

Also available: `install_error_hook()` to log uncaught exceptions, and `logging.getLogger().addHandler(JarvisLogHandler())` to log ERROR lines.

## Password and network access

The dashboard listens on `127.0.0.1` only and needs no password by default.

```bash
python jarvis_reports.py --set-password      # require a password
python jarvis_reports.py --remove-password   # turn it off again
python jarvis_reports.py --host 0.0.0.0      # other devices: a password is mandatory
```

## Configuration

| Variable | Purpose | Default |
|---|---|---|
| `ANTHROPIC_API_KEY` | Enables the AI Chat tab | none |
| `JARVIS_CHAT_MODEL` | Model used by AI Chat | `claude-sonnet-5-5` |
| `JARVIS_REPORT_DB` | Database file path | `jarvis_reports.db` next to the script |
| `JARVIS_REPORT_PASSWORD` | Password from the environment instead of the database | none |
| `JARVIS_SNAPSHOT_DAYS` | Days to keep CPU/RAM history | `30` |
| `JARVIS_ACTIVITY_DAYS` | Days to keep activities and finished tasks (`0` = forever) | `0` |
| `JARVIS_ALLOWED_HOSTS` | Extra host names when bound to a specific non-local address | none |

## Notes

- Data is stored in a local SQLite database (WAL mode).
- PDF export uses a built-in Courier font, so Urdu and Hindi text shows as `?` there. Use TXT or CSV for non-Latin text.
- All times are local to the computer running Jarvis.
