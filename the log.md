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

## Practicing git and GitHub push

Before touching the spirituality website, we ran a standalone exercise: turn the local `SWE` folder (holding these two log files) into a real git repository and push it to GitHub, deliberately separate from any new project so the mechanics could be learned without new backend concepts mixed in.

**The commands, in plain language, in the order run:**

1. `git init` — turns a plain folder into a tracked repository. Nothing is saved yet, this just switches on version tracking.
2. `git add .` — stages every file in the folder, marking it ready to be included in the next saved snapshot.
3. `git commit -m "..."` — actually saves that snapshot, with a message describing it.
4. `git branch -M main` — names the primary branch "main", GitHub's default name for it.
5. `git remote add origin <url>` — tells the local folder where its GitHub twin lives.
6. `git push -u origin main` — uploads the commit history to GitHub. The `-u` remembers this connection, so future pushes can just be `git push`, no need to repeat the full command.

**One real hurdle**: the first `git commit` failed with "Author identity unknown", git requires every commit to be attributed to someone, and this machine had never been told who that was. Fixed once, permanently, with:

```
git config --global user.email "lorenzocinquina5@gmail.com"
git config --global user.name "Lorenzo Cinquina"
```

This also explained a confusing earlier error: `git push` had failed with "src refspec main does not match any", which sounds like a connection problem but actually meant there was nothing committed yet to push, the failed commit upstream was the real cause.

**On authentication**: no manual token or password entry was needed. Git for Windows bundles a credential manager that automatically opened a browser login on the first push to a new GitHub remote. Approving it there was enough, git remembers the authorization for future pushes.

**End state**: `learning log.md` and `the log.md` are both live on GitHub at `github.com/lollocinq/swe-projects`, confirmed by viewing the repo in browser. This same six-command pattern (init, add, commit, branch, remote add, push) is what every future project will use to go from "written locally by Claude Code" to "live on GitHub."

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

---

# Project 2: Stillpoint — a spirituality website

A plain-language walkthrough of the second, much bigger project: a real web app with user accounts, an AI chat feature, a paywall, a live deployment, and an email newsletter. Where Project 1 was three small files living entirely inside GitHub's web editor, Project 2 is a proper application with dozens of files, a real database, and several outside services talking to each other. It's also where you started using Claude Code locally, in VS Code, with this chat providing context and checking its work rather than writing every file by hand here.

## What the product is, in one sentence

A small spirituality site where you register with just your email, chat for free with an AI guide inspired by Eckhart Tolle's teachings, pay a one-time €1 to unlock a second AI guide inspired by Jiddu Krishnamurti, and (optionally) get a weekly email with a reflective quote.

## Why build this, specifically

The site's content (spirituality, quotes, reflection) was almost beside the point. The real goal, agreed at the start, was to touch every major piece of how modern web apps actually work, deliberately, in one project: a real user database and login system, a third-party AI API, real payment processing with webhooks (built "the real way," not a shortcut), and a real deployment reachable by anyone, not just your own laptop.

## The full story, step by step

### 1. The idea and the layered build plan

Rather than building everything at once, the project was scoped into layers, each one adding exactly one new concept on top of a working product:

1. A static content site (just the look and pages, no backend yet).
2. Migrate that into a real framework (Next.js) with actual routing.
3. Add user registration (first real database, first real authentication).
4. Add the AI chat (first third-party API call, plus usage limits).
5. Add payment gating on the second guide (Stripe, webhooks, the deepest backend logic).
6. Deploy it live, publicly reachable (Vercel).
7. Add a newsletter (a second third-party API, Resend, plus a scheduled job).

Each layer was a working product on its own before the next one was added, the same "smallest real version first" instinct from Project 1, just applied to a bigger build.

### 2. From static HTML to Next.js

The site started as plain HTML/CSS pages, the same kind of thing Project 1's `index.html` was, just three pages instead of one. Once it needed real backend behavior (checking who's logged in, calling APIs, verifying payments), it was migrated into **Next.js**, a framework where the frontend (what you see) and backend (server-side logic) live in the same project instead of being two separate things glued together.

Next.js's **App Router** was the key idea to learn here: the folder structure of the project *is* the routing. A file at `app/presence-guide/page.js` automatically becomes the page at `yoursite.com/presence-guide`, no separate routing configuration file needed. The same pattern applies to backend endpoints: `app/api/send-newsletter/route.js` automatically becomes a URL, `/api/send-newsletter`, that can receive requests and run server-side code, without you wiring that connection up manually.

Within that, files are either **server components** (render on the backend before reaching the browser, can't respond to clicks) or **client components** (marked with `"use client"` at the top, run in the browser, can hold state and respond to interaction). Most of Stillpoint's interactive pieces, the auth forms, the chat windows, are client components.

### 3. Registration: Supabase and magic links

To let people register without you building a login system from scratch, the project uses **Supabase**, a hosted platform that bundles a real Postgres database with a ready-made authentication system, reachable through a JavaScript library rather than you writing SQL by hand for every basic operation.

Login uses **magic links** (Supabase calls the underlying method `signInWithOtp`): you type your email, Supabase emails you a one-time link, and clicking it logs you in. No password to create, remember, or leak. Under the hood this is proving you own that inbox, which is treated as good enough identity for a site like this.

Two Supabase API keys matter here, and mixing them up is a real security risk, not just a technicality:
- The **anon (publishable) key**: safe to expose in the browser, used for every normal logged-in action. It's still fully constrained by rules on the database side (see below), so even if someone inspected your website's code and found this key, they couldn't read or write data that isn't theirs.
- The **service role key**: bypasses every safety rule on the database. Used exactly once in this project, inside the Stripe webhook handler, because that code runs on the server with no logged-in user attached, and needs to be able to write "this user paid" on the user's behalf. This key never appears anywhere in the browser-visible code.

The database rule system that makes the anon key safe is called **Row Level Security (RLS)**: rules written directly on the database table itself (not just in your app's code) saying, in effect, "you may only see or change rows where the `user_id` column matches your own logged-in ID." So even a bug in the website's own code couldn't leak one user's data to another, the database itself refuses the request.

### 4. The AI chat: calling the Anthropic API

The "guides" are powered by calling Claude's own API (the Anthropic API) from the backend, the same underlying technology this very conversation runs on, just used programmatically instead of through a chat interface. Each guide's personality comes from a **system prompt**: a hidden instruction sent before the user's message, telling the AI how to behave. Here, the instruction was carefully worded to make each guide speak "inspired by" a teacher's general ideas, in its own voice, while explicitly never claiming to *be* that real person or presenting invented quotes as their actual words, an important, deliberate legal/ethical boundary, not an afterthought.

To stop the free guide being spammed (and to keep API costs bounded), a simple **rate limit** was added: a small database table records how many messages each user has sent that calendar day, checked before every new message, capped at five per day.

### 5. Payment gating: Stripe, checkout, and webhooks (the deep part)

This was the most involved piece of the whole project, deliberately: the brief for Project 2 specifically asked for payments to be built "the real way."

**Stripe Checkout** provides a ready-made, hosted payment page: rather than building your own credit card form (which brings serious security obligations), your backend asks Stripe to create a "Checkout Session" (a one-time payment request for a specific amount, here €1, tagged with metadata identifying which user is paying), Stripe hands back a URL, and the user is redirected there to actually enter card details on Stripe's own secure page.

The harder half is finding out *when* that payment actually succeeds, which is where **webhooks** come in. A webhook is the reverse of a normal API call: instead of your app repeatedly asking Stripe "has this been paid yet?", Stripe itself sends a request to a URL you provide the instant something happens (here, specifically the `checkout.session.completed` event). Your server just needs to be listening.

But a webhook URL is, by necessity, public and reachable by anyone on the internet, which raises an obvious question: what stops someone from just sending a fake "payment succeeded" request themselves? The answer is **signature verification**: every real webhook Stripe sends is cryptographically signed with a secret value only Stripe and your server know. Your server recalculates that signature from the raw request it received and compares it to the one Stripe sent; if they don't match, the request is rejected as untrusted. This required reading the incoming request body in its exact original, unprocessed form (not run through Next.js's usual automatic parsing), since the signature is calculated over those exact original bytes, any reformatting at all breaks the match.

For local testing, before the site was deployed anywhere, the **Stripe CLI** (a command-line tool, run as `stripe listen`) forwards Stripe's real webhook events to your own laptop, since Stripe's servers obviously can't reach `localhost` directly on their own. The important, easy-to-miss detail learned here the hard way: every time that command is restarted, it generates a brand-new signing secret. If the value saved in your environment file isn't updated to match, signature verification will fail on every single event, with no other symptom, a very confusing bug the first time you hit it (documented fully below).

Once a payment webhook is verified, the handler uses the Supabase **service role key** (from step 3) to write a row into a `payments` table recording that this specific user has paid, which the Krishnamurti Guide page then checks before unlocking its chat.

### 6. Going live: deploying to Vercel

Up to this point, everything only existed on your own laptop, reachable only while you had it running, exactly the "works on my machine" trap flagged as not-done back in Project 1. **Vercel** (built by the same team behind Next.js) is the hosting platform that turns the project into a real, always-on, public product: it connects directly to the GitHub repo, and every push to the `main` branch automatically triggers a fresh build and deployment, no manual upload step.

Two things needed re-doing for the live version specifically, both easy to forget:
- **Environment variables** (the Supabase keys, the Anthropic key, the Stripe keys) had to be re-entered into Vercel's own dashboard, since the local `.env.local` file that held them during development is deliberately never pushed to GitHub (it's listed in `.gitignore` precisely so secrets never leave your machine by accident).
- The **Stripe webhook** had to be pointed at the new live URL instead of `localhost`, which meant creating a second, permanent webhook endpoint in Stripe's own dashboard (separate from the temporary `stripe listen` one used for local testing) and saving *its* signing secret, a different value from the local one, into Vercel.

### 7. The newsletter: Resend and a scheduled job

The last piece reused the registered users' emails already collected during sign-up (rather than building a separate mailing-list signup form, judged not worth the extra complexity for this project's learning goals) and sends them a weekly email containing one rotating quote.

Sending it uses **Resend**, an email API purpose-built for apps sending mail programmatically (distinct from Project 1's Gmail SMTP approach, which suits a single personal alert but isn't the right tool for sending to many people from a real product). Triggering it on a schedule uses a **Vercel Cron Job**, defined in a small `vercel.json` file, conceptually the same idea as Project 1's GitHub Actions schedule, just built into the hosting platform directly this time. Vercel automatically attaches a secret authorization header to these scheduled requests, which the endpoint checks, so the same public URL can't be abused to trigger a mass email blast by anyone who finds it.

One real limitation, left in place on purpose rather than solved immediately: Resend restricts new accounts to only sending email to the address you signed up with, until you prove you own a real domain by adding specific DNS records it provides ("domain verification"). Since Stillpoint currently runs on a free `vercel.app` address rather than a purchased domain, the newsletter is fully built and working, but for now can only actually reach your own inbox. The code is already written to expand to every registered user automatically the moment a domain is verified, by flipping a single setting, no rebuild of the feature required.

---

## Debugging lessons earned on Project 2

This project had a much longer, harder debugging journey than Project 1, worth documenting honestly since these are exactly the kinds of bugs you'll meet again.

**The frozen browser tab.** Early on, opening the Krishnamurti Guide page froze the entire Chrome session, not just that tab. The cause: that page had created its own, second, independent Supabase client, separate from the one shared client used everywhere else in the app. Supabase's underlying login-tracking mechanism assumes exactly one client instance manages the browser's session lock; a second instance fighting over the same lock deadlocked. The fix was making sure literally every part of the app imports and reuses one single shared client, defined once in `lib/supabaseClient.js`.

**The bug that hid behind a fixed bug.** After fixing the freeze above, the page stopped crashing but silently always showed "not paid," even for accounts that had genuinely paid. The actual cause: a leftover chunk of old code, from before the client fix, was still referencing variables that no longer existed. It failed every time, but quietly, caught by a safety `try/catch` block designed to handle real errors gracefully, which meant there was no visible error at all, just a wrong-looking result. It only showed up by opening the browser's developer console directly and reading the actual underlying error message. Lesson: when something "just doesn't work" with no error on screen, that's often exactly when it's most worth checking the console anyway.

**Wrong column name.** A database query asked for a column, `id`, that the `payments` table never actually had (its primary key is a combination of `user_id` and `created_at` instead). Postgres refused clearly (`column payments.id does not exist`), an example of a database error that's actually the easy kind: specific and immediately explainable, once you know to go looking for it in the console.

**The one-character secret.** The single hardest bug in this whole project: the Stripe webhook kept failing signature verification, on every event, with no other clue than "signatures don't match." The eventual cause was almost invisible: the signing secret saved in the environment file had one extra repeated digit compared to the one actually shown in the running `stripe listen` terminal, a transcription slip made while first copying it in. Two long random-looking strings that look "close enough" at a glance are not close enough at all for a security signature; they either match exactly or the whole check fails with no partial credit and no helpful hint about which character is wrong.

**The platform's rule, not your bug.** When the newsletter route was first tested against the real Resend API, it failed with `403, "You can only send testing emails to your own email address"`. This wasn't a bug in the code at all, it was Resend's own sandbox restriction, present specifically because no domain had been verified yet. It then failed a *second* time, in a way that looked identical, after the obvious fix (limiting the recipient list) had already been applied, because a separate, easy-to-miss detail, the fixed "to" address on the email, also had to match the verified account address, not just the list of people being blind-copied. Reading the *exact* error message closely, twice, rather than assuming the first fix must have been complete, is what separated this from an actual code bug.

**Stale lock files from a mixed editing environment.** Because this project was edited both from your own local terminal *and*, at points, directly by this chat's own sandboxed environment, a `.git/HEAD.lock` file was accidentally left behind after one such edit, which then blocked your own next `git commit` with a "file exists" error until it was manually deleted. A reminder that git assumes only one process touches a repository at a time.

**Terminal aliasing.** Running `curl` inside PowerShell doesn't run the real curl at all, PowerShell secretly aliases that word to its own very different `Invoke-WebRequest` command, which expects headers in a completely different format and produces a confusing, unrelated-looking error. `curl.exe` explicitly, forces the real tool. A useful reminder that a command "not working" sometimes means a different program entirely answered to that name.

**Two separate meanings of "I can't do that for you."** Partway through, you asked directly whether some of the remaining setup steps could just be done automatically rather than manually. The honest answer split cleanly in two: editing files, running installs, and hitting public URLs could genuinely be done directly, from this chat's own sandboxed command-line access, proven by actually fixing `.env.local` and running `npm install` directly. But actions requiring *your* personal, already-logged-in credentials, pushing to *your* GitHub account, clicking inside *your* logged-in Vercel or Resend dashboards, could not, because that sandbox is a separate, isolated environment from your actual computer and doesn't share your logins. Even that sandbox later turned out to have its own further limits: a network allow-list blocked it from reaching your live `vercel.app` URL at all, which is why the very last verification test still had to be run by you, in your own terminal, even though almost everything before it had been done directly.

---

## New answers to questions you might get asked, about Project 2

### "What's actually protecting people's payment and account data here?"

Several independent layers, each covering a different risk: Supabase's Row Level Security means the database itself refuses to let one user's code see another user's rows, even before your app's own logic runs. The Stripe webhook's signature check means only genuine, cryptographically-signed events from Stripe are ever trusted, not just anything sent to that URL. And the most powerful key (the Supabase service role key, which bypasses those database rules) exists only on the server, inside one single file, never in anything the browser downloads.

### "Why did this take so much longer than Project 1?"

Project 1 had one data source, one action, one output. Project 2 has several independent outside services (Supabase, Anthropic, Stripe, Vercel, Resend) that all have to be configured correctly and agree with each other, plus real user accounts and real (if tiny) money. Most of the actual debugging time here wasn't writing new code, it was tracing exactly *which* of several connected systems a failure was coming from, which is genuinely representative of how larger real-world apps get built and maintained.

### "Is the €1 payment real money?"

During development, no, Stripe's "test mode" (test API keys, test card number `4242 4242 4242 4242`) simulates the entire flow with fake money for exactly this kind of building and testing. Switching to real payments later would mean swapping in Stripe's "live mode" keys instead, a deliberate, separate step, not something that happens by accident.

---

## The map of the new files added in Project 2

| File | Role |
|---|---|
| `app/page.js` | Home page: hero section and the rotating quotes. |
| `app/presence-guide/page.js` | The free guide's page (registration required, no payment). |
| `app/krishnamurti-guide/page.js` | The paid guide's page (registration + payment check, then chat). |
| `app/api/presence-chat/route.js` | Backend endpoint: verifies the user, enforces the daily message limit, calls the Anthropic API. |
| `app/api/create-checkout-session/route.js` | Backend endpoint: creates the Stripe Checkout Session for the €1 payment. |
| `app/api/stripe-webhook/route.js` | Backend endpoint: verifies Stripe's signed event, records a successful payment. |
| `app/api/send-newsletter/route.js` | Backend endpoint: gathers registered emails, picks a quote, sends via Resend. |
| `components/AuthGate.js` | Shared magic-link login form and session check, reused on both guide pages. |
| `components/Nav.js` | Top navigation bar, shows sign-out option when logged in. |
| `components/PresenceChat.js` | The chat interface component itself. |
| `lib/supabaseClient.js` | The one shared Supabase client instance, imported everywhere to avoid the multi-instance freeze bug. |
| `lib/quotes.js` | The shared list of quotes, used by both the home page and the newsletter. |
| `.env.local` | Local-only secrets (never pushed to GitHub): Supabase keys, Anthropic key, Stripe keys, Resend key. |
| `vercel.json` | The newsletter's weekly schedule, read by Vercel Cron. |

---

## New concepts you now know, in one line each (Project 2)

- **Framework**: a set of conventions and tooling built on top of a language that structures how you build something, rather than starting from a blank file every time.
- **File-based routing**: a project's folder structure directly determines its URLs, no separate routing configuration to maintain.
- **Client vs. server component**: renders in the browser and can react to clicks vs. renders ahead of time on the backend.
- **Magic link auth**: proving identity by clicking an emailed one-time link, no password involved.
- **Row Level Security**: database-enforced rules about who can read/write which rows, independent of your app's own code.
- **Anon key vs. service role key**: safe-to-expose, rule-constrained vs. secret, rule-bypassing.
- **System prompt**: a hidden instruction given to an AI before it sees the user's actual message, shaping how it responds.
- **Rate limiting**: capping how often something can be done in a given time window.
- **Checkout Session**: a one-time, hosted payment request that generates a page for the user to actually pay on.
- **Webhook**: a service proactively notifying your app the instant something happens, instead of your app repeatedly asking.
- **Signature verification**: cryptographically confirming a webhook request genuinely came from who it claims to, using a shared secret.
- **Vercel**: hosting that deploys a project automatically from GitHub and keeps it always-on and public.
- **Vercel Cron**: a scheduled trigger built into the hosting platform, this project's equivalent of Project 1's GitHub Actions schedule.
- **Transactional email API**: a service built specifically for apps to send email programmatically, as opposed to a personal inbox's SMTP.
- **Domain verification**: proving ownership of a domain via DNS records, required before an email service allows sending to arbitrary recipients.
- **Sensitive/write-only variable**: a secret value a dashboard lets you overwrite but never displays again once saved.
