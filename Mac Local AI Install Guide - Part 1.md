# Running Local AI on a Mac — Install Guide (Part 1)

*From the Work Play Have Fun video **"Running Local AI at Home — Why My £1,200 Mac Mini Went Back"**.*
*Tested in April 2026 on a Mac Mini M2 (16GB RAM). Last reviewed October 2026.*

---

## What this guide gets you

A private AI chatbot running entirely on your own Mac. Nothing you type leaves the machine, there's no subscription, and it still works when the online AI services go down.

This is Part 1: the simplest setup that works. **Ollama** (the free app that runs AI models) plus a few small models, all done in Terminal. It's what I set up in the video.

**Part 2** (coming with the next video) makes it usable by the whole family from their phones and laptops, through a browser.

> **A word of honesty before you start.** Local models are noticeably less capable than ChatGPT, Claude or Gemini, and their knowledge stops at their training date (some of mine were stuck in 2024). Privacy works brilliantly. Just don't expect a like-for-like replacement for the online models. Yet.

---

## What you need

| Item | Notes |
|---|---|
| A Mac with Apple Silicon (M1 or later) | I used a standard Mac Mini M2. Intel Macs will work, but slowly. |
| 16GB of memory (RAM) | Recommended. 8GB will run the smallest models only. |
| About 30GB of free disk space | For the models in this guide. Check in **System Settings > General > Storage**. |
| Your Mac's admin password | To install the app. |
| A decent internet connection | The models are big downloads (2–10GB each). |
| About an hour | Mostly waiting for downloads. |

**You do not need to buy new hardware to try this.** I bought a £1,200 Mac Mini M2 Pro first, then sent it back. My existing Mac did the job for nothing.

---

## Step 1 — Create a separate user profile (recommended)

I did all of this on a fresh user profile, kept apart from my normal one. If you decide local AI isn't for you, you delete the profile and everything goes with it.

1. Open **System Settings > Users & Groups**.
2. Click **Add User…** and enter your admin password.
3. Give it a name, e.g. *Local AI*, and a password.
4. Log out of your normal profile (**Apple menu > Log Out**) and log in to the new one.

---

## Step 2 — Get the Mac ready

1. **Update macOS:** **System Settings > General > Software Update**. Install anything waiting and restart if asked.
2. **Stop the Mac sleeping during the downloads:** **System Settings > Energy** (on a Mac Mini) and turn on **Prevent automatic sleeping when the display is off**. On a laptop, keep it plugged in.

---

## Step 3 — Install Ollama

1. Go to **https://ollama.com/download** and click **Download for macOS**.
2. Open the downloaded file and drag **Ollama** into **Applications**.
3. Open Ollama from Applications. Allow anything it asks for. A small llama icon appears in the menu bar at the top right of your screen.
4. Open **Terminal** (**Applications > Utilities > Terminal**) and type:

```bash
ollama --version
```

You should see a version number (mine was 0.20.7 in April; yours will be newer). If you get an error, click the llama icon in the menu bar to start Ollama and try again.

> Newer versions of the Ollama app also have their own chat window, which you can use instead of Terminal. I've stuck with Terminal here because that's what you see in the video, and it works the same on every version.

---

## Step 4 — A quick first test

Pull a small model (about 3.3GB):

```bash
ollama pull gemma3:4b
```

Then run it:

```bash
ollama run gemma3:4b
```

At the `>>>` prompt, ask it something, for example:

```
What is a local AI model and why would someone run one at home?
```

You should get a sensible answer within a few seconds. Type `/bye` to leave.

**If that worked, Ollama is installed and running.** That's the hard part done.

---

## Step 5 — Pull the models worth keeping

These are the models I tried on a 16GB Mac Mini M2. Pull them one at a time and try each before moving on.

```bash
ollama pull gemma4:e4b
ollama pull qwen3:8b
```

Run either of them the same way as before, e.g. `ollama run qwen3:8b`.

**Watch the spelling.** It's **qwen** with a **Q**. I typed "gwen" in the video and got `Error: pull model manifest: file does not exist`. Nine times out of ten, that error means a typo in the model name.

### What I found

| Model | Download | On a 16GB Mac Mini M2 |
|---|---|---|
| `gemma3:4b` | 3.3GB | Fast. Fine for a quick test and simple questions. |
| `gemma4:e4b` | 7–10GB | **Particularly good.** |
| `qwen3:8b` | 5.2GB | **Particularly good,** and still quick. A solid everyday model. |
| `qwen3:14b` | 9.3GB | Good answers, but **painfully slow and unworkable** on 16GB. Skip it unless you have 32GB. |
| `qwen2.5:32b` | 19GB | Only tried on the 32GB Mac Mini M2 Pro. **Won't fit on 16GB.** |

**Rule of thumb:** macOS keeps 4–6GB of memory for itself, so on a 16GB Mac you have roughly 10GB left for the model. Models of about 8 billion parameters (the "8b" in the name) are the sweet spot. Anything over about 14b won't fit.

> **Models move fast.** These were current in April 2026. Newer ones keep appearing (Qwen 3.5, for example, is out now) and are often better at the same size. Browse **https://ollama.com/search** and look for one in the 4b–9b range that fits your memory.

---

## Step 6 — Keeping an eye on disk space

See what you've got installed and how big each model is:

```bash
ollama list
```

Remove one you don't want:

```bash
ollama rm qwen3:14b
```

---

## Keeping it private and safe

- Everything runs on your Mac. Nothing you type is sent anywhere.
- Out of the box, Ollama only listens to your own Mac, not your home network. Leave it that way for now. Part 2 covers sharing it with the family safely.
- **Never** open Ollama's port (11434) to the internet on your router. It has no password protection.

---

## Changed your mind? Undoing it all

1. Log in to your normal profile.
2. **System Settings > Users & Groups**, click the **(i)** next to the *Local AI* profile, and choose **Delete User**. Choose to delete the home folder as well.
3. If you installed Ollama in a profile you're keeping instead: quit Ollama from the menu bar, drag it from **Applications** to the Bin, then delete the hidden `.ollama` folder in your home folder (in Finder, press **Cmd+Shift+.** to show hidden files). The models live in there, and that's what frees the disk space.

---

## What's next

In **Part 2** I make the local AI available to the whole family from their phones and laptops, through a proper chat page in the browser, with their own accounts. The guide for that will be added here when the video goes out.

---

*Work Play Have Fun — "Always be learning. Share knowledge. Try to have fun."*
*YouTube: https://youtube.com/@workplayhavefun · Website: https://www.workplayhavefun.com*

*Free to use and share. Provided as-is: follow it at your own risk, and check the official Ollama site if anything here has changed.*
