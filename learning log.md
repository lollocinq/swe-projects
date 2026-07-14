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
- **git init**: turns a plain folder into a tracked repository, starts its version history. Nothing is saved yet, it just switches tracking on.
- **git add**: stages files, marks them as ready to be included in the next saved snapshot. `git add .` stages everything in the current folder.
- **git commit**: actually saves the staged snapshot, with a message describing the change. Requires a configured identity first (`git config --global user.email` / `user.name`), set once per machine.
- **git branch -M main**: names the primary line of work "main", GitHub's default branch name.
- **git remote add origin \<url\>**: links your local folder to its GitHub twin, so git knows where to push to.
- **git push -u origin main**: uploads your committed history to GitHub. The `-u` remembers the connection so future pushes can just be `git push`.
- **Credential manager**: Git for Windows includes a credential manager that opens a browser login automatically on your first push to a new remote, no manual token needed.
- **"src refspec main does not match any"**: an error that means there's nothing committed yet to push, not a connection problem. Traced back to the commit silently failing earlier due to missing author identity.

### New concepts from Project 2 (Stillpoint)

- **Next.js**: a framework (a set of conventions and tooling built on top of a language, here JavaScript) for building web apps where the same project handles both the frontend (pages) and the backend (API routes), instead of them being two separate codebases.
- **App Router**: Next.js's file-based routing system. A file at `app/presence-guide/page.js` automatically becomes the page at `/presence-guide`. A file at `app/api/send-newsletter/route.js` automatically becomes a backend endpoint at `/api/send-newsletter`. The folder structure IS the routing, no separate config needed.
- **Client component vs server component**: a server component renders on the backend before the page ever reaches the browser (faster, but can't use browser-only features like clicking or state). A client component (marked with `"use client"` at the top of the file) renders in the browser and can respond to clicks, hold state, etc. Most interactive pieces of Stillpoint (chat, auth forms) are client components.
- **Supabase**: a hosted backend platform bundling a real Postgres database, user authentication, and auto-generated APIs, so you don't have to build a database server or auth system from scratch.
- **Magic link / passwordless auth**: instead of a password, the user gets a one-time login link emailed to them. Clicking it proves they own that email and logs them in. Supabase's `signInWithOtp` handles generating and verifying this.
- **Row Level Security (RLS)**: a Postgres feature where the database itself enforces who can read/write which rows, not just the application code. Our policies said "a user can only see/insert/update rows where `user_id` matches their own logged-in ID" (`auth.uid() = user_id`), enforced even if a bug in our code tried to do otherwise.
- **Anon key vs service role key**: two different Supabase API keys with very different power. The anon (publishable) key is safe to expose in the browser and is still fully constrained by RLS. The service role key bypasses RLS entirely and must never leave the server, used only inside our Stripe webhook to write payment confirmations on the user's behalf.
- **Environment variables (`NEXT_PUBLIC_` prefix)**: in Next.js specifically, only variables whose name starts with `NEXT_PUBLIC_` get bundled into the browser-visible code. Everything else stays server-only by default, an intentional safety line between "safe to expose" and "must stay secret."
- **Third-party API integration (Anthropic API)**: called Claude's own API from our backend to power the chat guides, with a system prompt (a hidden instruction the AI is given before it sees the user's message) telling it to speak "inspired by" a teacher's ideas without impersonating a real person.
- **Rate limiting**: capping how many times a user can do something in a given window (5 chat messages/day here), tracked with a small database table storing a count per user per day, checked before allowing each new request.
- **Stripe Checkout Session**: Stripe's hosted, pre-built payment page. Our backend creates a "session" (a one-time payment request with an amount and metadata), Stripe returns a URL, and we redirect the user there instead of building our own credit card form.
- **Webhook**: a way for one service (Stripe) to notify another (our app) the instant something happens, by sending an HTTP request to a URL we provide, rather than us having to repeatedly ask Stripe "did anything happen yet?"
- **Webhook signature verification**: because a webhook URL is publicly reachable, anyone could fake a request claiming "payment succeeded." Stripe signs every real webhook with a secret only we and Stripe know, and our server checks that signature before trusting the event. Requires reading the request body in its raw, unparsed form, since the signature is calculated over the exact original bytes.
- **Stripe CLI / `stripe listen`**: a local tool that forwards Stripe's real webhook events to your own laptop during development, since Stripe's servers can't reach `localhost` directly. Important gotcha we hit firsthand: it generates a brand new signing secret every time it's restarted, so the secret saved in your env file can silently go stale.
- **Vercel**: the hosting platform (made by the creators of Next.js) that takes a GitHub repo and runs it as a real, always-on, publicly reachable app, rebuilding automatically on every push to `main`.
- **Vercel Cron Jobs**: scheduled triggers defined in a `vercel.json` file, similar in spirit to the GitHub Actions scheduler from Project 1, but built into the hosting platform itself. Vercel automatically attaches an `Authorization: Bearer <CRON_SECRET>` header to these requests, which is what lets an endpoint trust that a request genuinely came from the schedule and not the public internet.
- **Resend**: a transactional email API (a service purpose-built for apps to send email programmatically, as opposed to Gmail SMTP, which was fine for Project 1's single-recipient alerts but doesn't fit sending many emails from a real app). Its sandbox mode, before you verify a domain you own, restricts every address on a send (including the "to" field, not just recipients) to the one email you signed up with.
- **Domain verification**: proving to a service like Resend that you actually control a domain, by adding specific DNS records it gives you. Required before an app can email arbitrary recipients under your own sender name, rather than being permanently limited to sandbox mode.
- **Sensitive/write-only environment variables**: some dashboards (Vercel included) let you mark a variable "Sensitive," after which the platform will never display its saved value again, only let you overwrite it. Looking "empty" when editing one is expected behavior, not a sign it was lost.
- **Sandboxed tool environments**: the AI's own command-line access in this chat runs inside its own isolated environment, not literally your PC's terminal. It can read and write your actual files (proven by fixing `.env.local` directly), but does not share your terminal's login sessions, so actions needing your personal credentials (`git push` to your GitHub account, browser-based dashboards) still had to be run by you, in your own terminal/browser.
- **React list keys**: React needs a unique `key` on each item in a list you render with `.map()`, to track which item is which across re-renders. Using something that can repeat (like an author name, once we had two quotes from the same author) causes silent bugs; using something guaranteed unique per item (the quote text itself) fixed it.

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

### Project 2: Stillpoint (spirituality website) — COMPLETE (v1)
- **Idea**: a small spirituality site with two AI "guides" (chat companions inspired by, not impersonating, real teachers), one free, one paywalled, plus a weekly quote newsletter.
- **Problem solved**: gave you a real, live product that deliberately touches every major layer of a modern web app, on purpose, as a learning vehicle: auth, database, third-party AI API, payments, webhooks, deployment, and email.
- **Three screens**: Home (rotating quotes), Presence Guide (free AI chat, registration required), Krishnamurti Guide (AI chat, registration + one-time €1 payment required).
- **Core data**: registered users (Supabase Auth), chat usage counts (rate limiting), payment records, subscriber emails (reused from registered users).
- **Stack**: Next.js (frontend + backend in one project) → Supabase (auth + Postgres database) → Anthropic API (the chat guides) → Stripe (checkout + webhooks) → Vercel (hosting + cron) → Resend (newsletter email).
- **Built in layers, each one a deliberate lesson**: static placeholder pages first → migrated into Next.js pages/routing → Supabase magic-link registration (first real database + auth) → Anthropic chat integration with daily rate limiting (first third-party API with usage control) → Stripe Checkout + webhook-verified payment gating (deepest backend logic, deliberately built "the real way" rather than shortcuts) → deployed live on Vercel with production environment variables and a live Stripe webhook endpoint → weekly newsletter via Supabase's user list, a Vercel Cron job, and the Resend API.
- **Status**: COMPLETE and live. Registration, both guides, payment gating, and the newsletter all confirmed working end-to-end on the real production URL, not just locally.
- **Live URL**: https://stillpoint-app-xi.vercel.app
- **Known, deliberate v1 limitations, left as-is by choice**: the Krishnamurti Guide currently reuses the Presence Guide's chat component and system prompt rather than having its own distinct persona (judged not worth the extra build for the learning value it would add); the newsletter can only email your own address until a real domain is bought and verified with Resend (currently deferred).

**New debugging concepts learned while building Project 2:**
- **Multiple client instances / connection deadlock**: creating a second, independent Supabase client in one page (instead of reusing one shared instance app-wide) caused the entire browser tab to freeze. Fixed by importing one shared client from a single file everywhere. Lesson: some libraries assume exactly one instance manages a given resource (here, the browser's auth lock), and quietly deadlock if you violate that.
- **Silent leftover code after an AI edit**: an edit that looked complete still referenced an old, now-undefined variable from before the fix. It failed silently (caught by a try/catch) and always defaulted to "not paid" with no visible error, until checked directly in the browser console. Lesson: when a fix "doesn't work" with no error shown, check whether an old code path is still partially there.
- **Exact-character secrets are exact**: a webhook signing secret that looked "basically the same" (off by one repeated digit) failed 100% of the time with no partial credit. Copy-paste, don't retype, whenever a secret must match another system exactly.
- **A tool's own sandbox restrictions look like your app's bugs**: Resend's sandbox mode rejected a real, correctly-coded request, twice, for two different reasons (the bcc list, then separately the "to" address) that had nothing to do with a bug in our code and everything to do with an unverified sending domain. Reading the exact error message (not just "it failed") is what separated a real bug from a platform restriction.
- **Terminal aliasing**: PowerShell's built-in `curl` is not the real curl, it's an alias for `Invoke-WebRequest` with different syntax. `curl.exe` forces the real one. A command that "should work" failing with a confusing, unrelated-looking error is sometimes just a naming collision, not a deeper problem.

---

## Tooling setup: git + GitHub push (complete)
- **What**: turned the local `SWE` folder into a real git repository and pushed it to a new GitHub repo (`swe-projects`), using Claude Code's terminal access.
- **Why**: standalone practice run for the git workflow (init → add → commit → push) before using it inside a real project, so the mechanics wouldn't tangle with new backend/API concepts later.
- **Hurdle hit and fixed**: first commit failed with "Author identity unknown", git didn't yet know who you are. Fixed with `git config --global user.email` / `user.name`, a one-time, per-machine setup. This also explained the confusing follow-on error ("src refspec main does not match any"): the push failed because there was nothing committed yet, not because of a bad connection.
- **Status**: COMPLETE. Both log files confirmed live on GitHub at `github.com/lollocinq/swe-projects`.

---

## Next steps
- Project 2 (Stillpoint) is complete and live. Everything below is optional follow-up, not required for the project to count as done.
- **Optional**: buy a real domain (~10-15 EUR/year) and verify it with Resend, to let the newsletter reach every registered user instead of just your own inbox. No code changes needed, just flip `RESEND_DOMAIN_VERIFIED=true` in Vercel once verified.
- **Optional**: give the Krishnamurti Guide its own distinct chat persona and API route, instead of reusing the Presence Guide's, if you want that extra layer of polish later.
- **Not yet decided**: what Project 3 will be.

**Note on logs**: both log files live directly on the Desktop (`SWE` folder) and Claude has direct read/write access to them here, so no separate sync step is needed day-to-day. They're now also pushed to GitHub (`swe-projects` repo) for backup/version history. Project 2's code lives in a separate repo, `stillpoint-app`, deployed live on Vercel.
