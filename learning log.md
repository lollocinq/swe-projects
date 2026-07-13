# AI-Native Product Builder: Learning Log

Started: July 11, 2026
Goal: Go from idea to live, deployed product using AI-assisted development, without becoming a full-time software engineer.

---

## Concepts covered so far

- **Frontend**: what the user sees and interacts with (buttons, screens, text).
- **Backend**: the program running elsewhere that receives requests and does the work.
- **Client vs server**: the client asks (your phone/browser), the server answers.
- **API**: the fixed, agreed set of requests a frontend is allowed to send to a backend.
- **Database**: where data is stored long term, so it survives after the app closes.
- **Deployment**: putting a project somewhere always-on and public, so it's a real product, not just code running on one laptop. "Works on my machine" is not done.
- **RSS feed**: a webpage formatted as structured data so a program can read it automatically.
- **Cron job / scheduler**: a rule that tells a computer to run code automatically on a set schedule, without a human triggering it.
- **GitHub Actions**: a free service that runs code on a schedule in the cloud. Used both as our scheduler and our deployment target for Project 1.
- **App password**: a separate, limited password Google generates for automated tools, so you never expose your real Gmail login.
- **Simple file-based database**: for small projects, a plain text/JSON file that the program updates can act as a lightweight, real database, before you need anything more complex.
- **npm (Node package manager)**: comes bundled with Node.js, installs and manages libraries/tools. `npm install -g X` installs a tool globally, usable from any folder, not just one project.
- **Execution policy**: a Windows/PowerShell security setting that blocks scripts from running by default, to stop unknown scripts from silently executing. `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` allows locally-written or trusted-signed scripts to run, for your user only.
- **npm install scripts / `allow-scripts`**: newer npm versions also block a package's own setup scripts by default until you approve them (`npm approve-scripts`), a second, separate safety layer from execution policy.
- **Git for Windows / bash**: Claude Code (and many dev tools) expect a Unix-style command layer (bash) even on Windows. Git for Windows provides this alongside version control. A missing dependency like this shows up as a runtime error, not an install error, worth remembering when troubleshooting later.
- **Absolute vs. relative paths**: `cd Desktop\SWE` only works if your terminal is already inside your user folder; it fails from `System32` or any other starting point. A full path (`cd C:\Users\lenovo\Desktop\SWE`) always works regardless of where the terminal opened.
- **PATH / needing a fresh terminal**: an already-open terminal window doesn't see changes an installer makes to your system (like adding `git` or `claude` to PATH, the list of folders Windows searches for commands). Closing and reopening the terminal reloads it.

---

## Tooling setup: Claude Code (complete)
- **What**: installed Claude Code, an AI coding tool that works directly inside your project folder in the terminal (reads/edits files, runs commands), instead of pasting code back and forth in a browser.
- **Why**: flagged back in Project 1 as the natural next step once a project needs local editing, testing, or scales past a few files.
- **Stack**: Node.js v24.18.0 (LTS) → npm 11.16.0 → `npm install -g @anthropic-ai/claude-code` → Git for Windows (bash dependency) → `claude` launched inside `~\Desktop\SWE`, logged in with Claude Pro.
- **Hurdles hit and fixed**: PowerShell execution policy blocked npm; npm's own `allow-scripts` gate blocked Claude Code's postinstall step; Claude Code needed Git Bash, not just PowerShell; wrong starting directory caused a false "folder not found."
- **Status**: COMPLETE. `claude` runs from inside the SWE project folder.

---

## Projects

### Project 1: Mercor News Watcher (in progress)
- **Idea**: Automatically detect and email any news mentioning Mercor.
- **Problem solved**: Removes manual checking, ensures early awareness of company news.
- **One screen**: Static status page showing last check time and alerts sent so far.
- **One piece of data**: A news article (headline, link, date).
- **Stack**: Google News RSS (source), GitHub Actions (scheduler + hosting for the automation), a text file (tracks already-sent articles), Gmail SMTP with app password (delivery), GitHub Pages (status page).
- **Status**: COMPLETE and live. GitHub Actions runs the checker every 3 hours, sends email via Gmail SMTP, and commits its own state back to the repo. Status page live on GitHub Pages, reading `status.json` client-side to show last-checked time and all alerts sent. Confirmed working end to end: real email received, real live webpage.
- **Live URL**: https://lollocinq.github.io/mercor-news-watcher/

**New debugging concepts learned while deploying:**
- **Exit code / permissions errors**: a workflow can fail partway through even if earlier steps (like sending the email) succeeded. Steps run in order, and a later failure doesn't undo earlier ones.
- **`permissions: contents: write`**: GitHub Actions defaults to read-only access to your repo for safety. Writing back to the repo (like our status file) needs this explicitly granted, scoped to just that workflow.
- **"Re-run jobs" vs "Run workflow"**: re-run replays the exact old commit that originally triggered the run, ignoring any fixes made since. "Run workflow" starts fresh from the current state of the branch. Use "Run workflow" whenever testing a fix.
- **Client-side data fetching**: a static page can call `fetch()` on a plain JSON file sitting next to it and render the results with JavaScript, no real backend server needed for something this simple. This is the same underlying idea as an API call, just aimed at a file instead of a server endpoint.
- **First-run backlog behavior**: when a dedup file starts empty, the very first run will treat all existing items as "new." Worth expecting this whenever a watcher/dedup system is reset or first deployed.

---

## Next steps
- Scope and start Project 2: a spirituality website (front end + back end, email capture, newsletter, payments/subscriptions), built in layers: content site live first, then email capture, then payments once there's a real audience.

**Note on logs**: both log files live directly on the Desktop (`SWE` folder) and Claude has direct read/write access to them here, so no GitHub sync is needed. Logs get updated in place at natural milestones or on request.
