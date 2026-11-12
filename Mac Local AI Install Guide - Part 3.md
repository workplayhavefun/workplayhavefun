# Running Local AI on a Mac — Install Guide (Part 3: web search with SearXNG)

*From the Work Play Have Fun video **"My Local AI Was Stuck in 2024 — Free Web Search With SearXNG"**.*
*Tested in April 2026 on a Mac Mini M2 (16GB RAM). Last reviewed November 2026.*

---

## What this guide gets you

In [Part 1](Mac%20Local%20AI%20Install%20Guide%20-%20Part%201.md) you got **Ollama** and a few models running on your Mac. In [Part 2](Mac%20Local%20AI%20Install%20Guide%20-%20Part%202.md) you put **Open WebUI** in front of it so the whole family can use it from a browser. But a local model only knows what was in its training data, so anything recent is a blank.

This part gives it the internet. You run **SearXNG**, a free, open-source search engine, in Docker on the same Mac, and point Open WebUI at it. When you ask something with web search turned on, Open WebUI searches, passes the results to the model, and the model answers from them. No API key, no account, no monthly limit.

> **An honest word first.** It works, but on a 16GB Mac it's slow. Searching, reading the results and then thinking about them takes a while, and in the video I could have searched it myself on my phone and had the answer first. A bigger model gives better answers but takes even longer.

### Why not Brave Search?

Older guides (mine included) say Brave's Search API has a free tier of 2,000 searches a month with no card needed. Brave dropped that free tier in February 2026: you now get $5 of credit a month and need a credit card. That's why this guide uses SearXNG instead.

---

## What you need

| Item | Notes |
|---|---|
| Parts 1 and 2 done | Ollama with at least one model, and Open WebUI running in Docker Desktop on port 3000. |
| Docker Desktop running | The whale icon in the menu bar. |
| About 15 minutes | Mostly one download. |

---

## Step 1 — Run SearXNG in Docker

Open **Terminal** and run:

```bash
docker run -d \
  --name searxng \
  -p 8080:8080 \
  -e SEARXNG_SECRET_KEY=changeme \
  --restart always \
  searxng/searxng
```

Replace `changeme` with any random string. If you'd like Terminal to make one for you, run `openssl rand -hex 32` and paste what it prints.

The first time, you'll see *"Unable to find image 'searxng/searxng:latest' locally"* and then a download. That's normal: Docker is fetching it.

---

## Step 2 — Check it's running

```bash
docker ps
```

You should see both **`open-webui`** (port 3000) and **`searxng`** (port 8080) in the list. Docker Desktop's **Containers** page shows the same.

You can also open **http://localhost:8080** in a browser on the Mac: the SearXNG search page should load.

---

## Step 3 — Let SearXNG answer in JSON (the fix you'll need)

Out of the box SearXNG only gives results as web pages (HTML). Open WebUI asks for **JSON**, and if SearXNG won't give it, every search ends with **"An error occurred while searching the web"**. In the video it found results and then threw that error. This step stops that, so do it now.

Check what it's set to:

```bash
docker exec searxng cat /etc/searxng/settings.yml | grep -A 5 "formats"
```

If you only see `- html` under `formats:`, add `json`:

```bash
docker exec -it searxng sh -c "sed -i 's/    - html/    - html\n    - json/' /etc/searxng/settings.yml"
```

Restart SearXNG:

```bash
docker restart searxng
```

Run the check again. You should now see:

```yaml
  formats:
    - html
    - json
```

---

## Step 4 — Connect Open WebUI to SearXNG

1. Open Open WebUI in your browser (**http://localhost:3000**) and sign in as the admin.
2. Go to **Admin Panel > Settings > Web Search**.
3. Turn **Web Search** on.
4. Set **Web Search Engine** to **searxng**.
5. In **Searxng Query URL**, enter:

   ```
   http://host.docker.internal:8080/search?q=<query>
   ```

6. **Save**.

> **Why `host.docker.internal` and not `localhost`?** Open WebUI runs inside Docker, and to it, `localhost` means its own container. `host.docker.internal` means "the Mac", where SearXNG is listening on port 8080.
>
> **About the `/search?q=<query>` part:** Open WebUI's current documentation says it's required. In April I typed just `http://host.docker.internal:8080` and that version worked, but use the full form.

---

## Step 5 — Try it

1. Start a **New Chat** and pick your model.
2. Turn on **Web Search** for the chat (in the options under the message box).
3. Ask something only the internet would know, for example:

   ```text
   Did the King of Britain visit any overseas countries this week?
   ```

You should see **Searching**, then **Searched 8 sites** (or similar), then **Retrieved 3 sources**, and then the model thinks about them and answers.

**What I found on a 16GB Mac Mini M2:**

| Model | With web search |
|---|---|
| Gemma 4 E4B | Much quicker. |
| Qwen3 14B | Slow (it only just fits in 16GB), but I expected it to be more accurate, so I went with it. |

It works from a phone too, the same as in Part 2: open your Mac's address on port 3000 in the phone's browser and turn web search on in the chat.

---

## If something's not right

| What you see | Try this |
|---|---|
| *"An error occurred while searching the web"* | Step 3 (the `json` format), then `docker restart searxng`. |
| It never finds anything | Check the URL in Step 4 (`host.docker.internal`, port `8080`, `/search?q=<query>`), and that `searxng` is in `docker ps`. |
| *403 Forbidden* or too many requests | That's SearXNG's built-in bot limiter. Open WebUI's SearXNG page explains how to relax it in `limiter.toml`. |
| It's painfully slow | That's a 16GB Mac doing three jobs at once. A smaller model is quicker; more memory is the real fix. |

---

## Changed your mind? Undoing it all

```bash
docker stop searxng
docker rm searxng
docker rmi searxng/searxng
```

Then turn **Web Search** off in **Admin Panel > Settings > Web Search**. Open WebUI, Ollama and your models are untouched.

---

*Work Play Have Fun — "Always be learning. Share knowledge. Try to have fun."*
*YouTube: https://youtube.com/@workplayhavefun · Website: https://www.workplayhavefun.com*

*Free to use and share. Provided as-is: follow it at your own risk, and check the official SearXNG and Open WebUI sites if anything here has changed.*
