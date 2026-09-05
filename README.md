# SteamGridDB Smart 1:1 Cover Downloader

A reliable Python tool for downloading **square (1:1) game artwork** from [SteamGridDB](https://www.steamgriddb.com/) using simple text-based game lists.

It is designed for large personal game libraries such as **PS4, PS5, Xbox One, and Xbox Series X|S**, with an emphasis on accurate game matching, square artwork, resumability, duplicate detection, retries, and organized output.

> **Important:** This project automates access to SteamGridDB. You are responsible for complying with SteamGridDB's current API policies/terms and for the rights applicable to any downloaded artwork.

---

## ✨ Features

### 🎮 Smart game-name matching

The downloader does **not** blindly trust the first autocomplete result.

It evaluates up to several search candidates using:

- Character similarity
- Substring matching
- Initialism matching
- Alternate names when supplied by the API
- A configurable minimum confidence threshold

If no candidate reaches the required confidence, the game is treated as **not found** instead of guessing.

The design intentionally prefers a missing cover over silently assigning the wrong game's cover.

### 🖼️ Square 1:1 covers

Only square artwork is accepted.

The preferred resolutions are:

```text
1024 × 1024
512 × 512
```

Other square resolutions may remain eligible depending on configuration.

Portrait Steam-style artwork is rejected when it is not square.

### ⭐ Artwork quality ranking

When multiple eligible grids exist, the downloader ranks them using several signals:

1. Square dimensions
2. `1024×1024` preference over `512×512`
3. SteamGridDB score when available
4. Artwork style
5. Static artwork preference
6. English metadata preference when available

### ⚡ Controlled parallel processing

Games are processed concurrently with a configurable worker count.

Default:

```python
MAX_WORKERS = 4
```

You can override it:

```bash
python downloader_final.py --workers 6
```

### 🔁 Automatic retries

Transient network/server failures are retried automatically.

Common retryable conditions include:

- Connection failures
- Read failures
- HTTP `429`
- HTTP `500`
- HTTP `502`
- HTTP `503`
- HTTP `504`

### 💾 Persistent state and resume

The program maintains:

```text
covers/_state.json
```

This keeps a persistent registry of processed games and their downloaded covers.

The state is periodically written during long runs and flushed again when processing exits.

### ♻️ Duplicate/shared-cover detection

The program builds an in-memory index of existing PNG covers.

This allows it to efficiently detect a cover that already exists elsewhere in the `covers/` directory without repeatedly scanning the entire output tree for every game.

### 📝 Failed-game logs

Each list gets its own:

```text
covers/<list-name>/_failed_games.txt
```

Typical reasons include:

```text
Not found in SteamGridDB
No valid square (1:1) cover found
Selected grid has no image URL
Download failed: ...
Network error: ...
```

### 🔄 Retry only failed games

Use:

```bash
python downloader_final.py --mode retry
```

to process only games currently present in the failure logs.

After a successful retry, that failure entry is removed.

### 🧪 Image validation

Downloaded images are inspected before being kept.

The program validates that:

- The response is not empty
- The file is not obviously too small
- The image format can be recognized
- Image dimensions can be read
- Width and height are equal

Common image headers supported by the validator include PNG, JPEG, GIF, and WEBP.

### 📦 Four operating modes

| Mode | Purpose |
|---|---|
| `full` | Process every game in the current lists |
| `retry` | Retry games recorded in failed logs |
| `missing` | Process games without a currently registered/local cover |
| `redownload` | Process every game again and fetch fresh artwork |

---

# 📦 Requirements

- Windows, Linux, or macOS
- Python **3.9+**
- Internet connection
- SteamGridDB API key
- Python package `requests`

No database server, Node.js, PHP, or web server is required.

---

# 🚀 Installation

## 1. Install Python

Download Python from:

https://www.python.org/downloads/

On Windows, enable:

```text
Add Python to PATH
```

Verify the installation:

```bash
python --version
```

On some systems:

```bash
python3 --version
```

---

## 2. Install Requests

Inside the project directory:

```bash
pip install requests
```

If `pip` is not available:

```bash
python -m pip install requests
```

Linux/macOS:

```bash
python3 -m pip install requests
```

---

# 📁 Project Setup

A minimal project can contain only:

```text
steamgriddb-downloader/
└── downloader_final.py
```

Run:

```bash
python downloader_final.py
```

The program creates the required directories when needed.

---

# 🗂️ Automatic list creation

If `lists/` does not exist, it is created automatically.

If no list files are found, the program creates:

```text
lists/
├── ps5.txt
├── ps4.txt
├── xbox_one.txt
└── xbox_sx.txt
```

The `covers/` directory is also created automatically.

Then:

1. Open the generated `.txt` files
2. Add your game names
3. Save the files
4. Run the downloader again

---

# 📝 Creating game lists

Use one `.txt` file per platform or collection.

Example:

```text
lists/
├── ps5.txt
├── ps4.txt
├── xbox_one.txt
└── xbox_sx.txt
```

Put **one game title per line**.

Example:

```text
Astro Bot
Black Myth: Wukong
Demon's Souls
God of War Ragnarök
Marvel's Spider-Man 2
Ratchet & Clank: Rift Apart
```

Blank lines are ignored.

Lines beginning with `#` are ignored as comments:

```text
# My PS5 collection

Astro Bot
Demon's Souls
Marvel's Spider-Man 2
```

---

# 🔑 SteamGridDB API Key

Open:

```text
downloader_final.py
```

Find:

```python
API_KEY = "YOUR_STEAMGRIDDB_API_KEY_HERE"
```

and replace it with your own key.

The application requires a valid API key before processing begins.

## ⚠️ GitHub warning

The personal-use version keeps the API key directly in the Python file.

**Never publish your real API key in a public repository.**

Before pushing to GitHub:

```python
API_KEY = "YOUR_STEAMGRIDDB_API_KEY_HERE"
```

If a real key has already been exposed publicly, revoke/rotate it.

---

# ▶️ Running the Downloader

Run:

```bash
python downloader_final.py
```

Without a mode argument, the program shows:

```text
1) Full Run      - Process everything
2) Retry Failed  - Retry only failed games
3) Only Missing  - Download missing covers
4) Re-download   - Re-download everything
```

---

# 🟢 First Run

For a new collection, choose:

```text
1
```

or run directly:

```bash
python downloader_final.py --mode full
```

The downloader will:

1. Read all text lists from `lists/`
2. Ignore blank lines and comments
3. Sort lists by number of games
4. Search SteamGridDB
5. Match the intended game
6. Fetch square grid artwork
7. Rank eligible covers
8. Download the selected image
9. Validate the image
10. Save the cover
11. Update persistent state
12. Record failures where necessary

---

# 🔄 Run Modes

## Full Run

```bash
python downloader_final.py --mode full
```

Processes all games in the current lists.

---

## Retry Failed

```bash
python downloader_final.py --mode retry
```

Processes games found in the failure logs.

Useful after fixing names or waiting out temporary network/API problems.

---

## Only Missing

```bash
python downloader_final.py --mode missing
```

Processes games that do not currently have a registered/local cover.

Useful when adding new titles to an existing library.

---

## Re-download

```bash
python downloader_final.py --mode redownload
```

Forces every listed game through the download workflow again.

Use this when you intentionally want to refresh existing artwork.

---

# ⚙️ Configuration

Important settings are near the top of `downloader_final.py`.

## Network timeouts

```python
SEARCH_TIMEOUT = 15
GRID_TIMEOUT = 20
DOWNLOAD_TIMEOUT = 30
```

## Retry behavior

```python
MAX_RETRIES = 3
RETRY_BACKOFF = 1.5
```

## Parallel workers

```python
MAX_WORKERS = 4
```

Command-line override:

```bash
python downloader_final.py --workers 6
```

## Request spacing

```python
JOB_DELAY = 0.15
```

The delay is applied inside workers before their actual API work.

## Search matching

```python
SEARCH_CANDIDATES = 5
MIN_MATCH_RATIO = 0.45
```

If no candidate reaches the minimum confidence threshold, the game is rejected rather than guessed.

## Cover sizes

```python
PREFERRED_SIZES = {(1024, 1024), (512, 512)}
ALLOW_OTHER_SQUARE_SIZES = True
```

---

# 🧠 Game Matching

Game discovery and artwork selection are two separate steps.

First, SteamGridDB autocomplete results are collected.

The downloader then scores candidate names instead of simply trusting the server's first result.

It considers:

### Character similarity

Useful for normal spelling variations and small typos.

### Substring matching

Useful for shortened names.

For example:

```text
witcher 3
```

can match a longer candidate containing those words.

### Initialisms

Useful for certain abbreviated names.

### Numeric abbreviations

Some glued forms such as numeric editions can match when the candidate contains the same numeric component as a separate word.

The matcher is deliberately conservative. It does not blindly convert arbitrary numbers to Roman numerals and does not fall back to a weak first search result.

---

# 🖼️ Artwork Selection

Once a game is identified, the downloader requests grid artwork.

Only square candidates are eligible.

A higher internal score is given to candidates with:

- `1024×1024`
- `512×512`
- Higher SteamGridDB score
- Preferred artwork styles
- Static artwork
- English metadata when available

The highest-ranked eligible grid is selected.

---

# ✅ Image Validation

The program validates the downloaded data before saving it permanently.

It rejects:

- Empty responses
- Obviously tiny files
- Unrecognized image data
- Non-square images

Images are first written to a temporary `.part` file and then moved into their final location.

---

# 📂 Output Structure

Example:

```text
steamgriddb-downloader/
│
├── downloader_final.py
│
├── lists/
│   ├── ps5.txt
│   ├── ps4.txt
│   ├── xbox_one.txt
│   └── xbox_sx.txt
│
└── covers/
    ├── _state.json
    ├── _run_log.txt
    │
    ├── ps5/
    │   ├── Astro Bot.png
    │   ├── Demon's Souls.png
    │   ├── Marvel's Spider-Man 2.png
    │   ├── _failed_games.txt
    │   └── _shared_games.txt
    │
    ├── ps4/
    │   ├── Bloodborne.png
    │   ├── God of War.png
    │   └── _failed_games.txt
    │
    └── xbox_one/
        └── ...
```

---

# 💾 Persistent State

The registry is stored in:

```text
covers/_state.json
```

Records can include:

- Official game name
- List name
- Local file path
- SteamGridDB game ID
- Selected cover URL
- Cover information
- Download timestamp

The state file is periodically flushed instead of being rewritten after every individual completion.

A final write is also performed when the processing section exits.

---

# ♻️ Duplicate and Shared Covers

The downloader builds a cover index once at startup.

If the same official game already has a cover elsewhere in `covers/`, it can reuse that cover instead of downloading another copy.

Shared-cover information is written to:

```text
covers/<list-name>/_shared_games.txt
```

The index is also updated during the current run so newly downloaded covers can be recognized by later jobs.

---

# 📝 Failed Games

Every list can have:

```text
covers/<list-name>/_failed_games.txt
```

Example:

```text
Game Name	Not found in SteamGridDB
Another Game	No valid square (1:1) cover found
Third Game	Download failed: HTTP 503
```

Use:

```bash
python downloader_final.py --mode retry
```

to retry the recorded failures.

---

# 📊 Progress and Final Report

During processing, the program displays:

- Completed/total jobs
- Current game
- Processing speed
- Estimated time remaining

Example:

```text
[OK] 120/5000 | Game Name | Speed: 1.48/s | ETA: 55m 12s
```

At the end:

```text
==============================================
              DOWNLOAD COMPLETED
==============================================
Downloaded      : 4700
Already exists  : 180
Shared covers   : 75
Failed          : 45
Total processed : 5000
Time            : 56m 31s
Workers         : 4
==============================================
```

A human-readable run summary is also appended to:

```text
covers/_run_log.txt
```

---

# 🛠️ Troubleshooting

## `ModuleNotFoundError: No module named 'requests'`

Run:

```bash
python -m pip install requests
```

---

## `python is not recognized`

Install Python and add it to PATH, then restart your terminal.

Check:

```bash
python --version
```

---

## `401 Unauthorized`

Check the API key configured in:

```python
API_KEY = "..."
```

---

## `403 Forbidden`

Check your SteamGridDB API access and credentials.

---

## `429 Too Many Requests`

You are being rate limited.

Try reducing:

```python
MAX_WORKERS = 2
```

and increasing:

```python
JOB_DELAY = 0.5
```

Then use:

```bash
python downloader_final.py --mode retry
```

instead of repeatedly running a full collection.

---

## A title is placed in `_failed_games.txt`

Check the recorded reason.

If the problem is the name itself, correct the `.txt` entry and run:

```bash
python downloader_final.py --mode retry
```

---

## The downloader does not guess a weak game match

This is intentional.

If the matcher cannot reach the configured confidence threshold, the title is failed rather than attaching a potentially incorrect cover.

---

# 🌐 GitHub Publishing

A public repository should normally contain only the source and documentation, for example:

```text
downloader_final.py
README.md
LICENSE
.gitignore
```

Avoid committing personal generated data such as downloaded covers and private game lists unless you intentionally want to publish them.

A suitable `.gitignore` includes:

```gitignore
__pycache__/
*.py[cod]

covers/
*.part

.venv/
venv/
env/

.idea/
.vscode/

Thumbs.db
.DS_Store
```

### 🔐 API key

Before publishing, remove your real SteamGridDB API key from the source.

Never commit credentials to a public repository.

---

# ⚖️ License

The source code may be distributed under the **MIT License** when the repository includes an MIT `LICENSE` file.

The source-code license does **not** automatically grant rights to third-party artwork downloaded from SteamGridDB.

SteamGridDB, artwork, game titles, logos, trademarks, and related assets remain subject to their respective owners and applicable terms.

---

# ⚠️ Disclaimer

This project is an automation tool and is not affiliated with SteamGridDB unless explicitly stated by the repository owner.

You are responsible for:

- API usage
- API credentials
- Downloaded artwork
- Redistribution of downloaded artwork
- Compliance with SteamGridDB's current policies and terms
- Applicable copyright and trademark requirements

---

# 🤝 Contributing

Contributions are welcome.

Potential improvements include:

- Better title normalization
- Additional artwork ranking signals
- More metadata caching
- Improved image validation
- More CLI options
- Automated tests
- Better reporting
- Additional platform presets

---

# ⭐ Project Philosophy

The project follows a simple rule:

> **A missing cover is better than silently assigning the wrong cover.**

The downloader therefore prioritizes:

**Accuracy → Reliability → Resumability → Efficiency**

rather than blindly maximizing request speed.

---

# 📚 Links

- SteamGridDB: https://www.steamgriddb.com/
- SteamGridDB API: https://www.steamgriddb.com/api/v2
- Python: https://www.python.org/
- Requests: https://requests.readthedocs.io/

---

## Current Version

```text
2.x
```

The project version refers to this downloader implementation and is independent of the SteamGridDB API version.
