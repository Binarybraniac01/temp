# MyStreamer - Torrent → Telegram → Stream Pipeline

Stream any torrent on your MyStreamer website using your phone as the downloader,
Telegram as free cloud storage, and Cloudflare Workers as a free streaming proxy.

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌────────────────────┐     ┌────────────┐
│  Your Phone │────▶│   Telegram   │────▶│ Cloudflare Worker  │────▶│ MyStreamer  │
│  (Termux)   │     │   Channel    │     │  (CORS Proxy)      │     │  Website   │
│             │     │  (Free ∞ GB) │     │  (Free ∞ bandwidth)│     │  (Vercel)  │
│ aria2c DL   │     │              │     │                    │     │            │
│ pyrogram UP │     │  Stores the  │     │  Extracts video    │     │  Plays the │
│             │     │  video file  │     │  from TG embed &   │     │  stream    │
└─────────────┘     └──────────────┘     │  streams with CORS │     └────────────┘
                                         └────────────────────┘
```

**Cost: $0** — Everything uses free tiers.

---

## ⚠️ Limitations

| Limitation | Details |
|---|---|
| **File size** | Telegram free: **2GB max** per file. Premium: 4GB. |
| **Upload speed** | Depends on your phone's internet connection |
| **CF Worker CPU** | 10ms CPU/request (free tier). Proxying video is fine. |
| **CF Worker requests** | 100,000 requests/day — more than enough for personal use |

---

## Setup Guide

### Step 1: Telegram Setup (5 minutes)

1. **Get API credentials:**
   - Go to [https://my.telegram.org](https://my.telegram.org)
   - Log in → "API Development Tools" → Create new application
   - Note down your **API_ID** and **API_HASH**

2. **Create a Bot:**
   - Open Telegram → search for **@BotFather**
   - Send `/newbot` → follow the prompts
   - Note down the **BOT_TOKEN** (looks like `123456789:ABCdefGHI...`)

3. **Create a Public Channel:**
   - In Telegram → New Channel → give it a name
   - Make it **PUBLIC** with a username (e.g., `mystreamer_files`)
   - Go to Channel Settings → Administrators → Add your bot as admin
   - Give the bot **"Post Messages"** permission

### Step 2: Cloudflare Worker (10 minutes)

1. **Create a free Cloudflare account** at [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)

2. **Option A: Deploy via Dashboard (easiest)**
   - Go to Workers & Pages → Create Application → Create Worker
   - Name it `mystreamer-proxy`
   - Click "Deploy" (it creates a hello-world worker)
   - Click "Edit Code" → paste the contents of `cloudflare-worker/index.js`
   - Click "Save and Deploy"
   - Your URL will be: `https://mystreamer-proxy.<your-subdomain>.workers.dev`

3. **Option B: Deploy via CLI**
   ```bash
   npm install -g wrangler
   wrangler login
   cd cloudflare-worker/
   wrangler deploy
   ```

### Step 3: Termux Setup on Phone (10 minutes)

1. **Install Termux** from [F-Droid](https://f-droid.org/packages/com.termux/)
   > ⚠️ Do NOT install from Google Play Store — that version is outdated and broken.

2. **Run the setup script:**
   ```bash
   # Copy the termux/ folder to your phone (via USB, ADB, or download from GitHub)
   # Then in Termux:
   cd ~/termux-torrent-stream/termux
   bash setup.sh
   ```

3. **Edit the config file:**
   ```bash
   nano config.py
   ```
   Fill in your `API_ID`, `API_HASH`, `BOT_TOKEN`, `CHANNEL_USERNAME`, and `CF_WORKER_URL`.

---

## Usage

### On your phone (Termux):
```bash
python torrent_to_tg.py "magnet:?xt=urn:btih:62bb50949c..."
```

The script will:
1. ⬇️ Download the torrent using aria2c
2. 📤 Upload the video to your Telegram channel
3. 🔗 Print a **Stream URL** like:
   ```
   https://mystreamer-proxy.your-name.workers.dev/stream/mystreamer_files/42
   ```

### On your website (MyStreamer):
1. Copy the Stream URL from Termux output
2. Paste it into the MyStreamer URL input
3. Click **Stream** → enjoy!

---

## Folder Structure

```
termux-torrent-stream/
├── termux/
│   ├── setup.sh            # One-time Termux dependency installer
│   ├── config.py           # Your API credentials (edit this!)
│   └── torrent_to_tg.py    # Main script: magnet → download → upload → URL
├── cloudflare-worker/
│   ├── index.js            # CF Worker: extracts & proxies Telegram video
│   └── wrangler.toml       # CF Worker deployment config
└── README.md               # This file
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `aria2c: command not found` | Run `pkg install aria2` in Termux |
| `pyrogram: No module named` | Run `pip install pyrogram tgcrypto` |
| Upload fails with "CHAT_WRITE_FORBIDDEN" | Make sure your bot is admin in the channel |
| CF Worker returns "Could not extract video" | Check that the channel is PUBLIC and message has a video |
| Video doesn't play in MyStreamer | The video may be MKV — browsers only play MP4/WebM natively |
| File too large (>2GB) | Split with `ffmpeg` or get Telegram Premium |
