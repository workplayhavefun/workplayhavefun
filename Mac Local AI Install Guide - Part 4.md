# Running Local AI on a Mac — Install Guide (Part 4: AI agents with AnythingLLM)

*From the Work Play Have Fun video **"Local AI Agents on a 16GB Mac Mini — Can AnythingLLM Do Real Jobs?"**.*
*Tested in May 2026 on a Mac Mini M2 (16GB RAM) with AnythingLLM 1.12.1. Last reviewed November 2026.*

---

## What this guide gets you

Parts 1–3 gave you a private chat AI: [Ollama](Mac%20Local%20AI%20Install%20Guide%20-%20Part%201.md) and its models, [Open WebUI](Mac%20Local%20AI%20Install%20Guide%20-%20Part%202.md) for the family, and [web search with SearXNG](Mac%20Local%20AI%20Install%20Guide%20-%20Part%203.md). This part tries something different: **AI agents** that do jobs, not just chat. Search the web and summarise it, make a spreadsheet you can download, write a small script, compare some apps and save the result as a file.

For that I used **AnythingLLM**, running in Docker alongside everything else and using the same Ollama models. Nothing is replaced: it sits next to Open WebUI on its own port, and you can remove it cleanly.

> **An honest word first.** It worked. Every job in the video came back done, the multi-step one in about 40 seconds. But on a 16GB Mac the best model I tried leaves only about 4GB for everything else, so don't expect to do much else on the Mac while it works.

---

## Which models to keep (what I found on 16GB)

I tested about 12 models and kept three:

| Model | Memory when loaded | Reads images? | Notes |
|---|---|---|---|
| Gemma 4 E4B | ≈ 10GB | Yes | Quick, good all-rounder. I used it for the agent tests. |
| Qwen 3.5 9B, Q8 (`qwen3.5:9b-q8_0`) | ≈ 12GB | Yes | By far the best reasoning and detail, but leaves only ≈ 4GB free. |
| Qwen3-VL 8B | ≈ 7.3GB | Yes | Smallest of the three. |

Delete the ones you won't use (`ollama rm <name>`): models take a lot of disk space, and on a 512GB Mac it adds up quickly. Check what's loaded and how much memory it takes with `ollama ps`.

---

## What you need

| Item | Notes |
|---|---|
| Part 1 done | Ollama running on the Mac, with at least one model. |
| Docker Desktop | From Part 2, running (the whale in the menu bar). |
| About 5GB free disk space | For the AnythingLLM image and its data. |
| About 20 minutes | Mostly the download. |

Check Ollama first, in **Terminal**:

```bash
ollama list
```

---

## Step 1 — Make a folder for AnythingLLM's data

So your workspaces, chats and settings survive a restart or an update:

```bash
mkdir -p ~/anythingllm-storage
touch ~/anythingllm-storage/.env
```

---

## Step 2 — Download and start AnythingLLM

```bash
docker pull mintplexlabs/anythingllm:latest
```

The image is a few GB, so give it a few minutes. Then start it:

```bash
docker run -d \
  --name anythingllm \
  -p 3001:3001 \
  --cap-add SYS_ADMIN \
  -v ~/anythingllm-storage:/app/server/storage \
  -v ~/anythingllm-storage/.env:/app/server/.env \
  -e STORAGE_DIR="/app/server/storage" \
  --restart unless-stopped \
  mintplexlabs/anythingllm:latest
```

| Part | What it does |
|---|---|
| `-p 3001:3001` | AnythingLLM on port 3001 (Open WebUI keeps 3000) |
| `--cap-add SYS_ADMIN` | Needed by some of the agent features |
| `-v ~/anythingllm-storage:…` | Keeps your data on the Mac |
| `--restart unless-stopped` | Starts again after a reboot |

Check it's running:

```bash
docker ps
```

You should see `anythingllm` with a status of **Up**.

---

## Step 3 — First launch and the set-up wizard

1. Open **http://localhost:3001** in your browser. If it won't connect, wait 30 seconds and try again. You should see **Welcome**: click **Get started**.
2. **LLM Preference:** choose **Ollama**. In the video it found Ollama and my three models straight away. Pick your model in **Ollama Model**.
   - If it can't find Ollama, open **Show advanced settings** and set the base URL to `http://host.docker.internal:11434`. Not `localhost`: inside Docker, `localhost` means the container itself.
3. **Data Handling & Privacy** should show **Ollama**, the **AnythingLLM Embedder** and **LanceDB**: your model, your documents and their search index all stay on your Mac.
4. Finish the wizard (create your account if it asks).

---

## Step 4 — Make a workspace and say hello

1. In the left sidebar, hover over the **+** and choose **New Workspace**. I called mine *Test*.
2. Type a simple message, for example *"Hello. How many people live in the UK?"*, and check the model answers.

---

## Step 5 — Switch on the agent skills

1. Open AnythingLLM's **Settings**, then **Agent Skills** (under *Instance Settings*).
2. Three are on already: **RAG & long-term memory**, **View & summarize documents** and **Scrape websites**.
3. Turn on the ones you want. I turned on:
   - **Web Search**: DuckDuckGo is built in and ready to use, no account or key.
   - **File System Access**: lets it read and write files.
   - **Document Creation**: lets it make spreadsheets and documents you can download.
   - **Generate charts**.
4. Click **Save** in the bar at the top.

> **A word on File System Access.** It lets the agent create, edit and move files. I ran all this under a separate macOS user profile (see Part 1), so nothing important was in reach. Do the same, or leave it off.

---

## Step 6 — Give it some jobs

Start a message with **`@agent`** to hand it over to the agent (type `/exit` to go back to normal chat). These are the jobs from the video:

**A web search**
```text
@agent What are the latest developments in local AI models in 2026? Search the web and summarise the key points.
```

**A spreadsheet you can download**
```text
@agent Create a simple spreadsheet template for tracking YouTube video ideas. Include columns for title, topic, status, and estimated publish date.
```
It made `youtube_video_ideas_tracker.xlsx` with a **Download** button.

**A small script**
```text
@agent Write a Python script that reads a CSV file and prints a summary of how many items are in each status category. Save the script as a .py file.
```

**A multi-step job**
```text
@agent Search the web for the top 5 most popular free video editing apps in 2026. Then create a comparison table as a markdown file and save it.
```
This took about 40 seconds with Gemma 4 E4B.

You'll see each step as it goes (*"Using DuckDuckGo to search for…"*, *"Model wants to call create-excel-file"*, *"Tool call was approved"*), so you can tell whether it actually used its tools.

---

## Using it from other devices

Like Open WebUI, AnythingLLM works from any phone, tablet or laptop on your home Wi-Fi: open `http://<your Mac's address>:3001`, using the same address as in Part 2 (e.g. `http://192.168.1.50:3001`).

---

## If something's not right

| What you see | Try this |
|---|---|
| Browser can't connect to `localhost:3001` | Wait 30–60 seconds after starting it; check `docker ps`; look at `docker logs anythingllm --tail 50`. |
| Can't find Ollama or no models listed | Is Ollama running (`ollama list`)? Use `http://host.docker.internal:11434`, then `docker restart anythingllm`. |
| The agent just chats and doesn't use its tools | Start the message with `@agent`, check the skills are on and **saved**, or try a different model: some are better at tools than others. |
| Port 3001 already in use | See what's using it: `lsof -i :3001`. |

---

## Handy commands

```bash
docker stop anythingllm               # stop it
docker start anythingllm              # start it again
docker logs anythingllm --tail 50     # see what it's doing
```

---

## Changed your mind? Undoing it all

```bash
docker stop anythingllm
docker rm anythingllm
docker rmi mintplexlabs/anythingllm:latest
rm -rf ~/anythingllm-storage          # deletes its workspaces, chats and settings
```

Ollama, your models, Open WebUI and SearXNG are untouched.

---

## What's next

In **Part 5** I give it the final test, a real question I needed answering, and decide whether a private AI workstation is worth the money.

---

*Work Play Have Fun — "Always be learning. Share knowledge. Try to have fun."*
*YouTube: https://youtube.com/@workplayhavefun · Website: https://www.workplayhavefun.com*

*Free to use and share. Provided as-is: follow it at your own risk, and check the official AnythingLLM and Docker sites if anything here has changed.*
