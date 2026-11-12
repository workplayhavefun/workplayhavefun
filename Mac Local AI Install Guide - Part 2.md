# Running Local AI on a Mac — Install Guide (Part 2: the whole family, in a browser)

*From the Work Play Have Fun video **"Local AI for the Whole Family — Can One Mac Mini Cope With Three at Once?"**.*
*Tested in April 2026 on a Mac Mini M2 (16GB RAM). Last reviewed November 2026.*

---

## What this guide gets you

In [Part 1](Mac%20Local%20AI%20Install%20Guide%20-%20Part%201.md) you got **Ollama** and a few models running in Terminal on your Mac. This part puts a proper chat page in front of it, **Open WebUI**, so anyone in the house can use your local AI from a phone, tablet or laptop browser, each with their own account. It still all runs on your Mac, and nothing leaves your home network.

> **An honest word first.** One person at a time works well on a 16GB Mac with a small model. Two or three people at once is a stretch: in the video, answers queued up and the Mac itself got laggy. The 14B model was too slow to be usable. Start small.

---

## What you need

| Item | Notes |
|---|---|
| Part 1 done | Ollama installed, with at least one model pulled (I used Gemma 4 E4B and Qwen3 8B). |
| Docker Desktop | Free for personal use. Open WebUI runs inside it. Allow a few GB of disk space for the two. |
| Your Mac's address on your home network | **System Settings > Network**, click your connection (Wi-Fi or Ethernet), **Details…** — it looks like `192.168.1.xxx`. |
| Phones / tablets on the **same** home Wi-Fi | For the family test. |
| About 30–45 minutes | Mostly downloads. |

---

## Step 1 — Install Docker Desktop

1. Go to **https://www.docker.com/products/docker-desktop/** and download the **Apple Silicon** version (M1, M2, M3, M4 Macs). Not the Intel one.
2. Open the download and drag **Docker** into **Applications**, then open it from there.
3. Accept the terms and allow what it asks for. You can skip signing in.
4. Wait until the whale icon in the menu bar stops animating: Docker is running.
5. Check it in **Terminal**:

```bash
docker --version
```

You should see a version number (mine was 29.4.0). Then:

```bash
docker ps
```

An empty list with column headings is fine: it means Docker is working and nothing is running yet.

---

## Step 2 — Install Open WebUI

Paste this into Terminal **exactly as it is** (all eight lines, in one go):

```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

The first time, it says *"Unable to find image … locally"*. That's normal: it then downloads Open WebUI from the internet. Give it a few minutes.

What the lines do, in plain English: run it in the background; show it on port **3000**; let it reach Ollama on your Mac; keep your accounts and chats in a storage area that survives restarts; call it *open-webui*; start it again automatically after a reboot.

Check it's running:

```bash
docker ps
```

You should see `open-webui` with a status of `Up`, on port 3000.

> **Tip from the video:** I was using a separate, locked-down user profile, so I couldn't copy and paste from my notes on the normal profile. If that's you, put the command in a text file on a USB stick, or type it carefully.

---

## Step 3 — First launch and your admin account

1. On the Mac, open a browser and go to **http://localhost:3000**
2. Click **Get started** and create your account. **The first account becomes the Admin**, which controls everything, so use a proper password, not a throwaway one.
3. Click **New Chat**, then the model name at the top: your Ollama models should be listed.
4. Pick one and ask it something. If an answer comes back, it's working.

**No models listed?** Make sure Ollama is running (the llama icon in the menu bar). Then in Open WebUI go to **Admin Panel > Settings > Connections** and check the Ollama address is `http://host.docker.internal:11434`.

---

## Step 4 — Try it from a phone or tablet

1. On a phone or iPad on the **same Wi-Fi**, open the browser and go to your Mac's address with **:3000** on the end, e.g. **http://192.168.1.131:3000** (use your own address).
2. Sign in with your admin account and ask a question.

On mine, the iPad answered quicker than the Mac's own browser, because the Mac was also busy running the screen recorder.

> **Tip:** on a phone, use the browser's **Share > Add to Home Screen** so it opens like an app.

---

## Step 5 — Accounts for the family

1. Click your profile picture > **Admin Panel > Users**, then **+** (Add User).
2. Give each person a name, email (it's just a login name here, nothing is sent) and password, with the role **User** (not Admin).

| Role | Can do |
|---|---|
| Admin | Everything: settings, models, users |
| User | Chat, pick a model, keep their own chats. No settings |

**The catch I hit in the video:** the new user signed in on my phone and saw *"No results found"* for models. New models are only visible to the admin until you share them. As the admin, go to **Admin Panel > Settings > Models**, open each model you want the family to use, and set its **access** to **Public**. Reload the page on the phone and the models appear.

---

## Step 6 — Pick models your Mac can cope with

What I found on a 16GB Mac Mini M2:

| Model | One person | Two or three at once |
|---|---|---|
| Gemma 4 E4B | Good, and quick enough | Slow: questions queue up |
| Qwen3 8B | Good, a bit slower | Laggy, but it got there |
| Qwen3 14B | Too slow to be usable | Don't |

- Each person's question waits for the one before. Three at once means three times the wait.
- Answers use the Mac's memory and processor, so the Mac itself gets sluggish while the AI is thinking. Video editing and AI at the same time was too much.
- To stop the family picking a slow model, set the ones you don't want back to private (Step 5), or switch them off: **Admin Panel > Settings > Models**, toggle the model off.

I left mine running on **Gemma 4 E4B** only.

---

## Keeping it private and safe

- It's on your **home network only**. Never forward port 3000 (or Ollama's 11434) on your router: anyone could find it.
- Away from home? Use a VPN back into your home network (I use WireGuard), then go to the same address.
- Your chats are stored on your Mac, inside Docker's `open-webui` storage area. Nothing goes to the cloud.
- Ollama's own port (11434) has no password. Leave it listening on your Mac only, as it is out of the box. Open WebUI reaches it from inside Docker without opening it to the network.

---

## Handy commands

```bash
docker ps                      # is Open WebUI running?
docker restart open-webui      # restart it
docker stop open-webui         # stop it (it starts again when Docker starts)
```

Update Open WebUI to the latest version (your accounts and chats are kept):

```bash
docker pull ghcr.io/open-webui/open-webui:main
docker stop open-webui && docker rm open-webui
```

then run the `docker run` command from Step 2 again.

---

## What's next

In **[Part 3](Mac%20Local%20AI%20Install%20Guide%20-%20Part%203.md)** I give the local AI the internet: free web search with SearXNG, running in Docker on the same Mac.

---

## Changed your mind? Undoing it all

```bash
docker stop open-webui
docker rm open-webui
docker volume rm open-webui        # deletes all accounts and chats
docker rmi ghcr.io/open-webui/open-webui:main
```

Then drag **Docker** from **Applications** to the Bin. Ollama and your models from Part 1 are untouched. (If you used a separate user profile, deleting the profile removes it all.)

---

*Work Play Have Fun — "Always be learning. Share knowledge. Try to have fun."*
*YouTube: https://youtube.com/@workplayhavefun · Website: https://www.workplayhavefun.com*

*Free to use and share. Provided as-is: follow it at your own risk, and check the official Open WebUI and Docker sites if anything here has changed.*
