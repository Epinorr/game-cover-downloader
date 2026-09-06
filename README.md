# SteamGridDB Smart 1:1 Cover Downloader

A reliable Python tool for downloading **square (1:1) game artwork** from [SteamGridDB](https://www.steamgriddb.com/) using simple text-based game lists.

Built for large personal game libraries such as **PS4, PS5, Xbox One, and Xbox Series X|S** — with accurate game matching, square-only artwork, resumability, safe duplicate handling, retries, and organized per-list output.

> **Important:** This project automates access to SteamGridDB. You are responsible for complying with SteamGridDB's API policies/terms and for the rights applicable to any downloaded artwork.

---

## ✨ Features

### 🎮 Smart game-name matching
The downloader never blindly trusts the first autocomplete result. It scores several search candidates using character similarity, substring matching, initialisms (e.g. `re4` → *Resident Evil 4*), and alternate names, then picks the best match above a configurable confidence threshold.

If nothing reaches that threshold, the game is reported as **not found** — the tool never guesses and silently attaches a possibly wrong cover. A missing cover you can fix is always better than a wrong one you won't notice.

### 🖼️ Square 1:1 covers only
Preferred resolutions: `1024×1024`, then `512×512`. Other square sizes stay eligible depending on config; non-square artwork is always rejected.

### ⭐ Artwork quality ranking
Eligible grids are ranked by resolution, SteamGridDB score, artwork style, static-vs-animated, and English metadata — the highest-ranked one is downloaded.

### ⚡ Controlled parallel processing
Configurable worker count (default `4`), overridable with `--workers N`. Per-worker request pacing keeps API usage polite without stalling job dispatch.

### 🔁 Automatic retries
Transient failures (connection issues, `429`, `500–504`) are retried automatically with backoff.

### 💾 Persistent state and resume
`covers/_state.json` tracks every processed game **per list** — the same game title in two different lists is tracked and downloaded independently, so one list's progress never gets confused with another's. State is flushed periodically during a run (not after every single game) and always flushed on exit, including on Ctrl+C or a crash.

### ♻️ Space-saving duplicate detection
If the same official game already has a cover downloaded somewhere else in `covers/`, the tool reuses it instead of downloading a second copy — backed by an in-memory index so this check stays fast even on huge libraries. Every reuse is logged centrally (see below) so you always know exactly where the real file lives.

### 📝 Centralized shared-cover report
Instead of a separate log buried in every list folder, all shared-cover events are collected in one place:
- `covers/_shared_covers_report.txt` — raw log of every reuse event.
- `covers/_shared_covers_summary.txt` — generated at the end of each run, grouped by real file, showing exactly which lists need a copy of which cover.

### 📝 Failed-game logs
Each list gets its own `covers/<list-name>/_failed_games.txt` with the reason (not found, no valid square cover, download error, etc.). Use `--mode retry` to reprocess only these.

### 🧪 Image validation
Every downloaded file is checked before being kept: not empty, not suspiciously small, a recognizable format (PNG/JPEG/GIF/WEBP), and square. Files are written atomically (temp file + rename) so a crash mid-download never leaves a corrupt cover in place.

### 🖥️ Stays open when done
The console window doesn't just close after finishing. It prints a final summary and waits for `Enter` before exiting — including after an unexpected error, so nothing flashes by unread. This is skipped automatically when run non-interactively (scripts, CI).

### 📦 Four operating modes

| Mode | Purpose |
|---|---|
| `full` | Process every game in the current lists |
| `retry` | Retry only games recorded in failed logs |
| `missing` | Process only games without a currently registered/local cover |
| `redownload` | Force-refresh every game's artwork |

---

## 📦 Requirements

- Windows, Linux, or macOS
- Python **3.9+**
- Internet connection + SteamGridDB API key
- `requests` package

No database, Node.js, PHP, or web server needed.

---

## 🚀 Installation

```bash
pip install requests
```

(or `python -m pip install requests` if `pip` isn't directly available)

---

## 📁 Setup

A minimal project needs only:

```text
steamgriddb-downloader/
└── downloader_final.py
```

Run it once:

```bash
python downloader_final.py
```

Required folders (`lists/`, `covers/`) are created automatically. If no list files exist yet, sample ones are generated for you (`ps5.txt`, `ps4.txt`, `xbox_one.txt`, `xbox_sx.txt`) — add your games and run again.

---

## 📝 Creating game lists

One `.txt` file per platform/collection, one game per line:

```text
lists/ps5.txt
------------------
# My PS5 collection
Astro Bot
Demon's Souls
Marvel's Spider-Man 2
```

Blank lines and lines starting with `#` are ignored.

---

## 🔑 API Key

Open `downloader_final.py` and set:

```python
API_KEY = "YOUR_STEAMGRIDDB_API_KEY_HERE"
```

A valid key is required before processing begins.

---

## ▶️ Running

```bash
python downloader_final.py
```

Without `--mode`, an interactive menu lets you pick Full / Retry / Missing / Re-download. Or specify directly:

```bash
python downloader_final.py --mode full
python downloader_final.py --mode retry
python downloader_final.py --mode missing
python downloader_final.py --mode redownload
python downloader_final.py --mode full --workers 6
```

---

## ⚙️ Configuration

Key settings near the top of `downloader_final.py`:

```python
# Concurrency
MAX_WORKERS = 4
JOB_DELAY = 0.15          # per-worker pacing between requests

# Search matching
SEARCH_CANDIDATES = 5
MIN_MATCH_RATIO = 0.45    # below this, a game is "not found" rather than guessed

# Cover sizes
PREFERRED_SIZES = {(1024, 1024), (512, 512)}
ALLOW_OTHER_SQUARE_SIZES = True

# Retry behavior
MAX_RETRIES = 3
RETRY_BACKOFF = 1.5

# State-file write frequency
STATE_SAVE_INTERVAL = 5.0  # seconds; always flushed on exit regardless
```

---

## 🧠 How matching works

Discovery and artwork selection are separate steps. First, SteamGridDB autocomplete candidates are collected; the tool then scores each one against your list entry using character similarity, substring containment (`witcher 3` matches *The Witcher 3: Wild Hunt*), and initialisms (`fifa23` matches *FIFA 23*). The highest-scoring candidate above `MIN_MATCH_RATIO` wins.

This is deliberately conservative: it won't convert arbitrary digits to Roman numerals, and it never falls back to a weak first result just to avoid a "not found." A wrong cover is harder to notice and fix than a missing one.

---

## 📂 Output structure

```text
steamgriddb-downloader/
├── downloader_final.py
├── lists/
│   ├── ps5.txt
│   └── xbox_one.txt
└── covers/
    ├── _state.json
    ├── _run_log.txt
    ├── _shared_covers_report.txt
    ├── _shared_covers_summary.txt
    ├── ps5/
    │   ├── Astro Bot.png
    │   └── _failed_games.txt
    └── xbox_one/
        └── ...
```

---

## 📊 Progress and final report

Live progress per game:

```text
[OK] 120/5000 | Game Name | Speed: 1.48/s | ETA: 55m 12s
```

Final summary (printed, and appended to `covers/_run_log.txt`):

```text
==============================================
                 JOB FINISHED
==============================================
Downloaded      : 4700
Already exists  : 180
Shared covers   : 75
Failed          : 45
Total processed : 5000
Total time      : 56m 31s
==============================================

Press Enter to exit...
```

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: requests` | `python -m pip install requests` |
| `401 Unauthorized` | Check your `API_KEY` |
| `403 Forbidden` | Check your SteamGridDB API access |
| `429 Too Many Requests` | Lower `MAX_WORKERS`, raise `JOB_DELAY`, then re-run with `--mode retry` |
| Game in `_failed_games.txt` | Check the logged reason; fix the list entry if needed, then `--mode retry` |
| A weak/ambiguous name isn't matched | Intentional — see [How matching works](#-how-matching-works) |

**Before publishing:** remove your real API key from the source. Never commit credentials to a public repository.

---

## ⚖️ License & Disclaimer

Source code may be distributed under the **MIT License** (if a `LICENSE` file is included) — this does **not** grant rights to third-party artwork downloaded from SteamGridDB. Artwork, titles, logos, and trademarks remain subject to their respective owners.

This is an independent automation tool, not affiliated with SteamGridDB. You're responsible for your API usage, credentials, downloaded artwork, and compliance with SteamGridDB's current terms and applicable copyright/trademark law.

---

## 📚 Links

- [SteamGridDB](https://www.steamgriddb.com/) · [API docs](https://www.steamgriddb.com/api/v2)
- [Python](https://www.python.org/) · [Requests](https://requests.readthedocs.io/)
