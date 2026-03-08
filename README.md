<p align="center">
  <a href="README.md" style="text-decoration: none;">
    <img src="https://img.shields.io/static/v1?label=&message=ENGLISH&color=212121&style=flat-square&logo=google-translate&logoColor=blue" alt="English" height="28"/>
  </a>
  <a href="README_AM.md" style="text-decoration: none;">
    <img src="https://img.shields.io/static/v1?label=&message=AMHARIC&color=212121&style=flat-square&logo=google-translate&logoColor=red" alt="Amharic" height="28"/>
  </a>
</p>

<p align="center">
  <img src="./hero/akasha_hero.png" width="100%" alt="Project Akasha Hero" />
</p>

<h1 align="center">
  <span>PROJECT AKASHA</span>
</h1>

<p align="center">
  <span align="center">A modular Telegram UserBot built on telethon. integrates Gemini LLM for context-aware automation, Edge-TTS for localized voice synthesis, and a custom media stack.</span>
</p>
<p align="center">
  <a href="#-installation">Installation</a>
  <span> · </span>
  <a href="#system-core">System Core</a>
  <span> · </span>
  <a href="#-modules-manual">Modules Manual</a>
  <span> · </span>
  <a href="#-troubleshooting">Troubleshooting</a>
</p> 

<p align="center">
  <img src="./hero/image.png" width="50%" alt="Project Akasha Tools" />
</p>

## 𓏵 Overview

**Project Akasha** solves the problem of "dumb" auto-replies. Unlike static userbots, it uses a **Neural engine** (Google Gemini) to generate responses that actually fit the conversation context, mimicking your typing style.

it includes a **TTS Wrapper** for amharic/english voice notes, a **Two-Stage Music Downloader** to save RAM, and various administration tools. The codebase is structured for easy deployment on local machines or cloud instances (Heroku/Railway).

## ➜ Project Structure

ensure your directory matches this tree. The bot relies on relative paths for fonts and database persistence.

```text
.
├── .env                  # API keys & configuration
├── config.py             # settings loader
├── main.py               # event loope entry point
├── requirements.txt      # python dependencies
├── docker-compose.yml
├── Dockerfile            # docker container configuration
├── Procfile              # cloud deployment instructions
├── setup.sh              # linux VPS automated setup
├── LICENSE               # Legal Usage Permissions
├── core
│   └── database.py       # JSON/Mongo handler
└── plugins
    ├── admin_tools.py    # group management
    ├── ai.py             # LLM & vision Logic
    ├── creative.py       # pillow-based image manipulation
    ├── master_voice.py   # edge-TTS & FFmpeg filters
    ├── music.py          # yt-dlp audio wrapper
    ├── security.py       # TTL capture
    └── system.py         # auto-reply logic
```

## ⚙ Installation

follow these steps to deploy. we use **environment variables** to prevent credential leaks.

<details open>
<summary><strong>1. Environment Prerequisites</strong></summary>
<br/>

Required: **Python 3.9+** and **FFmpeg** (for audio conversion).

**Ubuntu/Debian:**
```bash
sudo apt update && sudo apt install python3 python3-pip ffmpeg -y
```

**Windows:**
1. Install Python from python.org.
2. Install FFmpeg and add it to System PATH.
</details>

<details>
<summary><strong>2. Install Dependencies</strong></summary>
<br/>

```bash
pip install -r requirements.txt
```
</details>

<details>
<summary><strong>3. Configuration (.env)</strong></summary>
<br/>

Create a `.env` file in the root directory. copy the structure below.
**Note:** the bot supports API key rotation to bypass free tier rate limits. You can use a single list (`GEMINI_KEYS`) or individual variables (`GEMINI_KEY1`...).

```ini
# ---telegram core---
# get from my.telegram.org
API_ID=123456
API_HASH=your_api_hash
SESSION=1BVts... # telethon session

# --- AI engine ---
# method 1: single list
# Separate keys with commas (,)
GEMINI_KEYS=AIzaSy1...,AIzaSy2...,AIzaSy3...

# Method 2: individual variables
GEMINI_KEY1=AIzaSyD...
GEMINI_KEY2=AIzaSyF...
GEMINI_API_KEY=AIzaSy... # fallback

# --- database ---
# leave empty for local JSON
MONGO_URL=
```

> **⚠︎ security:** never commit your `.env` or `SESSION` string to public repositories.

</details>

<details>
<summary><strong>4. Execution</strong></summary>
<br/>

```bash
python main.py
```
*Triggers `Config.check_integrity()` on startup to validate keys.*

</details>

<br/>

## ☁︎ Cloud Deployment

for Render, Railway, or Heroku, skip the `.env` file. Inject these variables directly into the dashboard's **environment settings**:
*   `API_ID`
*   `API_HASH`
*   `SESSION`
*   `GEMINI_KEYS` (paste all your keys here, separated by commas. Example: `Key1,Key2,Key3`).

<div id="system-core"></div>

## 𖡎 System Core

managed by `plugins/system.py`. handles the "Away State" logic.

### Auto-Pilot (`.auto`)
controls the reply engine.

| Command | Logic |
| :--- | :--- |
| `.auto ai` | **neural mode:** fetches context from the last 5 messages and generates a response via Gemini. |
| `.auto static` | **static mode:** replies with a pre set string (e.g., "Busy"). |
| `.auto off` | **passthrough:** disables all automation. |
| `.auto [text]` | updates the static mode variable. |

### Context Modes (`.mode`)
injects specific system prompts into the AI to alter tone.
*   `sleep` - short, groggy responses.
*   `work` - professional, concise.
*   `gaming` - dismissive/short.
*   `default` - standard conversational style.

### Latency Simulation
to prevent bot detection:
1.  **read delay:** random `1-3s` sleep before marking as read.
2.  **typing:** calculates `len(response) * 0.1s` to simulate human typing speed.

***

## ⊞ Modules Manual

### 1. Admin Tools (`plugins/admin_tools.py`)
standard group administration.

*   **Whois:** `.whois @user`. generates a user profile card (ID, DC, Scam Status, Bio).
*   **Translator:** `Text //lang_code`. replaces the message with the translation (e.g., `Hey //am` -> `ሰላም`).
*   **moderation:**
    *   `.purge`: deletes messages recursively.
    *   `.ban / .mute`: standard user restrictions.
    *   `.zombies`: Scans for deleted accounts and cleans them to fix member counts.

> [!CAUTION]
> **Rate Limits:** massive purges or zombie cleaning can trigger telegram's `floodWait`. use sparingly.

---

### 2. TTS engine (`plugins/master_voice.py`)
direct wrapper for **Microsoft Edge TTS**. supports SSML tags for pitch/rate control.

**Command:** `.say [text] [flags]`
*   **auto detect:** switches between `Mekdes` (Amharic) and `Jenny` (English) based on scrip.

**Flags (FFmpeg Filters):**
| Flag | Filter Applied |
| :--- | :--- |
| `.f / .m` | Force Female/Male model. |
| `.echo` | `aecho` filter (Hall effect). |
| `.radio` | High-pass/Low-pass filter chain. |
| `.demon` | Pitch shift `-400Hz`. |
| `.kid` | Pitch shift `+400Hz`. |

*Shortcut:* `.whisper` applies `.slow` + low volume.

---

### 3. Image Utils (`plugins/creative.py`)
pillow based image manipulation.

*   **Meme Gen:** `.meme [Top];[Bottom]`.
    *   *logic:* auto-fetches `NotoSansEthiopic-Bold` or `NotoSansArabic` if the text script requires it. caches fonts locally.
*   **Sticker Kang:** `.kang`. converts media to `512px` WebP and appends to your sticker pack.
    *   *Limit:* Skips files >5MB to prevent memory leaks on VPS.

---

### 4. Music Loader (`plugins/music.py`)
uses `yt-dlp` for soundCloud/youtube extraction.

**Workflow:**
1.  **Search:** `.song [Query]`. fetches metadata onlys. Caches result in RAM (`SEARCH_STATE`).
2.  **Select:** Reply `1 to 5`.
3.  **Process:** Downloads -> converts to **MP3 192kbps** (FFmpeg) -> writes id3 tags -> uploads.

> [!NOTE]
> **Hard Limit:** files >50MB are rejected to avoid telegram upload timeouts.

---

### 5. AI & Vision (`plugins/ai.py`)

*   **Generative:** `.ai [prompt]`. uses the rotated API key pool.
*   **Vision:** reply to an image with `.ai explain`. Downloads image to memory buffer -> sends to Gemini Vision API.
*   **Image Search:** `.img` or `.imgs` (Gallery). scrapes DuckDuckGo for images.

---

### 6. Security Modules (`plugins/security.py`)

#### **TTL Capture (anti-view-once)**
**status:** `disabled` by default.
*   **function:** hooks into the `MessageMedia` event. If `ttl_period > 0`:
    1.  Downloads media to temp.
    2.  Forwards to "Saved Messages".
    3.  Appends "ꗃ vault capture" tag.

#### **Fake Terminal**
*   `.hack @user`: edits a message repeatedly to simulate a terminal breach. purely prank.

---

## 🔧 Troubleshooting

**1. FFmpeg Error:**
`FileNotFoundError`: ensure Ffmpeg is in your System PATH or installed via `apt`.

**2. API Key Limits:**
If `.ai` fails, check if your Gemini keys are valid. the bot auto rotates, but if all are exhausted, it will return an error.

**3. Dependency Issues:**
force reinstall: `pip install --force-reinstall -r requirements.txt`.

<br/>
<p align="center">
  <a href="https://t.me/cipher_attacks">
    <img src="https://img.shields.io/badge/TELEGRAM-CHANNEL-000000?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram">
  </a>
  <a href="https://github.com/Cipher-attack">
    <img src="https://img.shields.io/badge/SOURCE-CODE-000000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub">
  </a>
  <a href="#">
    <img src="https://img.shields.io/badge/STATUS-ACTIVE-00FF00?style=for-the-badge&logo=statuspage&logoColor=black" alt="Status">
  </a>
</p>

<p align="center">
  Licensed under the <a href="./LICENSE">MIT License</a>.
</p>

<p align="center">
  <b>Project Akasha v1.0</b><br>
  <i>Built for the Unknown purpose ☕︎.</i>
</p>