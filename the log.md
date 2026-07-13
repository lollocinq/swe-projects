# The Log: What We Actually Built and How

A plain-language explanation of the Mercor News Watcher project. Written so you can re-read it, explain it to others, and navigate the tools on your own.

---

## What the product is, in one sentence

A small robot that checks Google News for anything about Mercor every 3 hours, emails you when something new appears, and keeps a public webpage showing everything it has found so far.

---

## The full story, step by step

### 1. The idea and the spec

We started by scoping the smallest real version of your idea. Not "a news platform", just: one source (Google News), one trigger (a new article mentioning Mercor), one action (email you), one screen (a status page). Scoping small is what made it shippable in a day.

### 2. Where the code lives: GitHub

GitHub is a website where code projects live. Each project is a "repository" (repo): a folder that GitHub tracks over time. Every change you save is a "commit", a snapshot with a message describing what changed. Your repo is `mercor-news-watcher`.

When your dad asked how you did it and you said "GitHub", you were partially right, but GitHub is the *where*, not the *how*. The more complete answer: "I wrote a small Python program, and I use GitHub to store it, run it automatically on a schedule, and host its webpage."

### 3. The language: Python

The core logic is written in Python, one of the most popular programming languages, known for reading almost like English. Our whole product is one Python file, `check_news.py`. It does four things every time it runs:

1. Loads the memory file (`status.json`) to see what it already alerted you about.
2. Fetches the Google News RSS feed for "Mercor".
3. Compares: anything in the feed not already in memory is "new".
4. If there's anything new: emails you, then saves the updated memory.

### 4. The data source: an RSS feed

RSS is an old, reliable web format: a page structured for programs to read instead of humans. Google News offers its search results as RSS for free. That's how we get news without paying for a news API or scraping websites.

To read that feed we used `feedparser`, a Python "library". A library is pre-built code someone else wrote that you borrow instead of writing yourself. The file `requirements.txt` lists which libraries your project needs, so any computer running it knows what to install first.

### 5. The memory: status.json

A program forgets everything when it finishes running. To avoid emailing you the same articles forever, it needs persistent memory. Ours is `status.json`: a plain text file (in JSON format, a standard way of writing structured data) listing every article already sent plus a "last checked" timestamp.

This is a real database concept in its simplest form. Big products use dedicated database servers; for a project this size, a file is genuinely the right tool, not a shortcut.

### 6. The automation: GitHub Actions

This is the piece that makes it a product instead of a script. GitHub Actions is a free service where GitHub lends you a fresh computer in their data center to run your code, either on a schedule or on demand.

The file `.github/workflows/check_news.yml` is the instruction sheet for that computer:
- **When** to run: every 3 hours (the `cron` schedule, written in UTC time), plus a manual "Run workflow" button for testing.
- **What** to do: install Python, install the libraries, run `check_news.py`, then commit the updated `status.json` back to the repo so the memory persists to the next run.

This is why your laptop being off doesn't matter. The product runs on GitHub's machines, not yours. That's what "deployed" means.

### 7. The secrets: Gmail app password + GitHub Secrets

To send email through your Gmail, the script needs credentials. Two safety mechanisms keep this from being dangerous in a public repo:

- **App password**: a special 16-character password Google generates just for this tool. It's not your real password and can be revoked anytime without affecting your account.
- **GitHub Secrets**: encrypted storage on GitHub where the credentials live. The workflow injects them into the script at runtime as "environment variables" (values a program receives from the system running it, rather than having them written in the code). They never appear in your code, which is why the repo can safely be public.

### 8. The email: SMTP

SMTP is the standard protocol (agreed-upon language) email servers use to send mail. Python has `smtplib` built in, so the script logs into Gmail's mail server with your app password and sends the message. Later we upgraded the email to HTML format (styled like a tiny webpage) for the cheese theme. One quirk learned: email clients only respect styles written directly on each element, no shared stylesheets like real webpages use.

### 9. The website: GitHub Pages

GitHub Pages is another free GitHub feature: it takes files in your repo and serves them as a real public website. Our `index.html` is the entire frontend: structure (HTML), styling (CSS, including the cheese-hole dotted background), and behavior (JavaScript).

The JavaScript does one job: `fetch("status.json")` asks for the memory file sitting in the same repo, then builds the article list on screen from that data. This is the same idea as an API call, just pointed at a plain file instead of a server, because our data is simple enough not to need a real backend server answering requests.

So the same `status.json` powers both channels: the email (push, comes to you) and the website (pull, you go to it).

---

## Answers to the questions you'll get asked

### "How did you actually build this?"

"I wrote a Python script that reads Google News' RSS feed, deduplicates against a JSON file, and emails me via Gmail. It runs every 3 hours on GitHub Actions, and GitHub Pages hosts a status page reading the same data." That sentence is 100% accurate and you now understand every word in it.

### "Why didn't we use Claude Code or Cursor?"

Claude Code and Cursor are AI coding *environments*: tools where an AI reads and edits files directly on your computer, runs commands, and works with you inside a code editor. They're excellent, and you'll likely use them for bigger projects.

We didn't need them here for two reasons:
1. The project is 3 small files. Writing them in GitHub's own web editor was simpler than setting up a local development environment (installing Python, an editor, git, etc. on your machine).
2. It kept everything in one place. You never had code "on your machine" at all, which conveniently sidestepped the whole "works on my machine" trap. The code was born deployed.

The tradeoff: editing files one at a time in a web browser doesn't scale. When a project has 20 files, or needs testing before committing, you'll want a local setup with an AI coding tool. That's a natural next step, not something we skipped by mistake.

### "Is this a real product or just a script?"

Real product. The test isn't size, it's: does it run continuously, without you, somewhere always-on, doing something useful, reachable by others? Yes on all counts. Plenty of paid SaaS products are architecturally this exact shape: scheduled job + notification + status page.

### "What did it cost?"

Zero. GitHub (repos, Actions, Pages) is free at this scale, Google News RSS is free, Gmail is free. The only money ever discussed was an optional custom domain (~10-15 EUR/year), which we haven't bought.

### "Where does the code run?"

On GitHub's servers (Microsoft's data centers, since GitHub is owned by Microsoft), on a fresh temporary Linux computer that spins up every 3 hours, does the job in about 20 seconds, and disappears. The only thing that survives between runs is what's committed to the repo, which is exactly why the workflow commits `status.json` back.

---

## The map of every file in the repo

| File | Role |
|---|---|
| `check_news.py` | The brain. Fetches news, dedupes, sends email, updates memory. |
| `requirements.txt` | Shopping list of libraries to install (just `feedparser`). |
| `.github/workflows/check_news.yml` | The schedule and instructions for GitHub's computers. |
| `status.json` | The memory. Written by the script, read by the website. |
| `index.html` | The website: structure, cheese styling, and the data-fetching script. |
| `README.md` | The repo's homepage text. |

---

## Concepts you now know, in one line each

- **Frontend / backend**: what you see vs. the machinery behind it.
- **Client / server**: the one asking vs. the one answering.
- **API**: the fixed set of requests a program is allowed to make of another.
- **Database**: persistent memory that survives after the program stops. Ours is a JSON file.
- **Deployment**: code running somewhere always-on and public, not on your laptop.
- **Repository / commit**: a tracked project folder / a saved snapshot of changes.
- **Library / requirements.txt**: borrowed pre-built code / the list of what to borrow.
- **RSS feed**: a webpage formatted for programs, not people.
- **Cron / scheduler**: "run this automatically at these times."
- **GitHub Actions**: free borrowed computers that run your code on a schedule.
- **GitHub Pages**: free hosting that turns repo files into a public website.
- **Secrets / environment variables**: how credentials reach code without living in it.
- **App password**: a revocable, limited password for automated tools.
- **SMTP**: the protocol for sending email.
- **HTML / CSS / JavaScript**: a webpage's structure / looks / behavior.
- **JSON**: a standard text format for structured data.
- **Regex**: a pattern for finding/cleaning text (we used one to strip HTML tags).
- **UTC**: the neutral global timezone schedules run on. Italy is UTC+2 in summer.

---

## Setting up Claude Code

Project 1 was built entirely through GitHub's web editor, no code ever touched your actual laptop. That worked because the project was three small files. It stops working once a project has many files or needs local testing before committing, which is exactly the situation Project 2 (the spirituality website) will be. So the first real task of Project 2 wasn't the website itself, it was getting Claude Code running locally.

**What Claude Code is**: an AI coding tool that runs inside your terminal, sitting directly in your project folder. Unlike chatting with Claude in a browser, it can read your actual files, edit them, and run commands itself, install a library, run a script, check for errors, without you copying anything back and forth.

**The install chain**: Claude Code is distributed as an npm package (npm is Node.js's package manager, the tool that installs and manages other tools/libraries). So the dependency order was: install Node.js (which includes npm) → use npm to install Claude Code globally (available from any folder, not just one project) → run `claude` inside the project folder.

**Three real obstacles, each a small lesson in how Windows protects itself:**

1. **PowerShell execution policy.** `npm -v` failed immediately after installing Node, because Windows PowerShell blocks scripts from running by default, and npm's command is technically a small script. Fixed with `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`, which allows locally-written or trusted scripts to run, scoped to your user only, not a security downgrade for the whole machine.

2. **npm's own script-approval gate.** After installing Claude Code, npm separately refused to run the package's own setup script until approved (`npm approve-scripts --allow-scripts-pending`). This is a newer, independent safety layer from the execution policy, worth knowing they're two different gates.

3. **Missing bash dependency.** Once `claude` was runnable, it refused to start from inside the project folder, because Claude Code expects a Unix-style command layer (bash) even on Windows, which plain PowerShell doesn't provide. Fixed by installing Git for Windows, which bundles Git Bash. This also means the machine is now set up with Git itself, useful later for working with the repo locally instead of only through GitHub's web editor.

A smaller, non-technical snag along the way: `cd Desktop\SWE` failed at first because the terminal had opened inside `C:\WINDOWS\System32`, which has no Desktop folder. Using the full path (`cd C:\Users\lenovo\Desktop\SWE`) fixed it. Relative paths only work relative to wherever the terminal happens to already be.

**End state**: `claude` launches successfully from `~\Desktop\SWE`, logged in with your Claude Pro account, ready to read and edit files in that folder directly.

---

## Debugging lessons already earned

- Workflows run step by step; a late failure doesn't undo earlier steps (you got the email even when the run "failed").
- GitHub Actions is read-only on your repo by default; writing back needs `permissions: contents: write` in the workflow file.
- "Re-run jobs" replays the old commit. "Run workflow" uses the current code. When testing a fix, always use "Run workflow".
- A watcher's first run against empty memory reports everything as new (the 100-article email). Expected, one-time behavior.
- Reading the Actions log is the single most useful debugging habit: it shows exactly which step failed and why.
- Windows has layered, independent safety gates (execution policy, npm script approval) that can each block a working install at a different stage. An error doesn't mean the install failed, just that one gate needs an explicit yes.
- A tool refusing to *run* (not install) often means a missing dependency it silently assumed you'd have, like Claude Code assuming bash. The fix is usually installing that dependency, not reinstalling the tool itself.
- Always use full paths (`C:\Users\...`) when unsure where a terminal session started. `cd` failures are often just a wrong starting point, not a missing folder.
