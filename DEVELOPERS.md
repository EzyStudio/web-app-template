# EzyStudio Web App template — developer guide

This repository is the **Web App template** (`astro-app`) that every EzyStudio app site starts
from: Astro and a Hono backend on one Cloudflare Worker, its own sign-in and roles, a React
dashboard and a Postgres database. It is published for developers who build **web app listings
for the [EzyStudio App Store](https://ezystudio.com/app-store)** on their own computer. You build a
whole site on this template, submit it, and every customer who installs it gets a new site made
from your code.

This guide is for you, the developer. It never reaches a customer's site: submit leaves it out.
`README.md` is the template's own setup guide (it ships into every site, so it never names us);
`AGENTS.md` and `docs/agent/` are the rules for the AI that works on the site, yours first, then the
customer's.

**License:** see [`LICENSE`](LICENSE). You may use and change this template to build sites and apps
that run on EzyStudio; the code you write stays yours.

## Contents

1. [Quick start](#1-quick-start)
2. [Clone and remotes](#2-clone-and-remotes)
3. [Your admin account](#3-your-admin-account)
4. [Build your app](#4-build-your-app)
5. [Check it like submit does](#5-check-it-like-submit-does)
6. [Submit](#6-submit)
7. [What review checks](#7-what-review-checks)
8. [Your public preview](#8-your-public-preview)
9. [How updates work](#9-how-updates-work)
10. [Troubleshooting](#10-troubleshooting)
11. [FAQ](#11-faq)
12. [For your AI](#12-for-your-ai)
13. [Reporting problems and suggesting changes](#13-reporting-problems-and-suggesting-changes)

Links: [developer docs](https://ezystudio.com/developers/docs) ·
[developer program](https://ezystudio.com/developers) ·
[developer portal](https://ezystudio.com/dev) ·
[the Website template](https://github.com/EzyStudio/website-template) (static sites)

## 1. Quick start

You need Node.js 22.12+, npm, git and Docker.

```bash
git clone https://github.com/EzyStudio/web-app-template.git my-app && cd my-app
git remote rename origin upstream
npm install
docker compose up -d --wait db
npm run setup
npm run dev                                   # http://localhost:4321
```

Sign up at http://localhost:4321/account/register/, type the code printed by `npm run dev`, then
`npm run dev-admin -- you@example.com` and open http://localhost:4321/dashboard/.
Every step in detail: `README.md`.

## 2. Clone and remotes

```bash
git clone https://github.com/EzyStudio/web-app-template.git my-app
cd my-app
git remote rename origin upstream             # the template: releases come from here
git remote add origin git@github.com:<you>/<your-repo>.git
git push -u origin main
```

- **Clone; don't press "Use this template" and don't copy the files.** A clone keeps the template's
  history, so each release merges into your work with one `git merge` ([How updates work](#9-how-updates-work)).
- **Watch releases:** on https://github.com/EzyStudio/web-app-template press **Watch → Custom →
  Releases**. Each release notes what changed and whether it is required. Submit always needs the
  template's current version.
- Started from the zip in the developer portal? See [From a zip](#from-a-zip).

## 3. Your admin account

There is no Site Key on your computer, so the site runs in **local development mode**: account
emails (sign-up codes, reset links) are printed where `npm run dev` runs instead of being sent.

1. Sign up at http://localhost:4321/account/register/ and type the 6-digit code from the terminal
   (`npx astro dev logs` when the dev server runs in the background).
2. `npm run dev-admin -- you@example.com` makes that account an admin in your LOCAL database.
3. Open http://localhost:4321/dashboard/.

Nothing makes you the owner locally: on a customer's site the owner is the customer, through "View
as admin". Owner-only actions (editing roles) can't be tried locally.

## 4. Build your app

- Read `AGENTS.md` and `docs/agent/` first, and have your AI read them before every task.
- Your code: `src/server/`, `src/pages/`, `src/portal/`, `src/content/`, `src/lib/`, `src/styles/`,
  `prisma/`. **Never change** `src/platform/`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `docs/agent/`
  or `template.json`: submit compares them with the template byte for byte.
- Describe your app in `README.md`: what it does, where each feature lives, what an owner usually
  changes. The customer's AI reads it.
- Migrations only add (tables, nullable columns, indexes); never edit a released one.
- Postgres only, no secrets in any file, no install scripts, no deploy config.
- Rules in full: [Building an app](https://ezystudio.com/developers/docs/building-apps),
  [Security rules](https://ezystudio.com/developers/docs/security),
  [Database](https://ezystudio.com/developers/docs/database),
  [Best practices](https://ezystudio.com/developers/docs/best-practices).

**Sample data, `prisma/seed.sql`** (optional): loaded into the review test site and your public
preview, and into a customer's **staging** database only when they press "Load sample data" —
never their live one. Wrap it in `BEGIN;` … `COMMIT;`, stop with `RAISE EXCEPTION` when real data
is there, make it safe to run twice, use `@example.com` addresses and no users. The template's own
seed is the example. [Sample data rules](https://ezystudio.com/developers/docs/database#seed).

## 5. Check it like submit does

Download the developer kit from [Templates](https://ezystudio.com/dev/templates) in the developer
portal. Copy its `store-check.mjs` and `STORE-LISTING.md` (every store rule in plain words) into your
repository, add both names to your `.gitignore`, then:

```bash
node store-check.mjs                          # the same checks as submit
npm run check                                 # tests and the production build
```

## 6. Submit

1. **Join the developer program** at https://ezystudio.com/developers. Once approved, your team
   works in the [developer portal](https://ezystudio.com/dev).
2. Push your branch to GitHub.
3. Open [Submit an app](https://ezystudio.com/dev/submit): name, category, description and data
   note; screenshots and icon; under **App code**, connect GitHub and pick the repository and
   branch. The checks run on its latest commit at once and show the first problem with its file;
   after a push, press **Check again**.
4. Pricing: free, one-time or monthly (you keep 80%).
5. Send it for review. Submitting freezes that commit.

[Submitting](https://ezystudio.com/developers/docs/submitting) has the details.

## 7. What review checks

Levels, one after another, each with the file and the fix when it fails:
package rules (current template version, `src/platform/` and the AI instructions untouched) →
security (secrets, code that reads sign-ins or keys, vulnerable packages, install scripts, outside
payments, other databases) → build (a real test site with your migrations and sample data) →
performance → UI and accessibility → data and privacy → an AI-assisted code review → a human
reviewer who opens the test site. [Review](https://ezystudio.com/developers/docs/review).

## 8. Your public preview

Once approved, the store hosts a preview of your app built from the approved code, with its own
database and your sample data, so people can try it before installing. In the developer portal:
- **Get admin login** emails each person on your team an admin login for the preview (never owner).
- **Load sample data again** empties the preview and loads your seed again.

## 9. How updates work

Two kinds of update meet in your repository:

- **Template releases** (from `upstream`): tagged `vX.Y.Z`, with release notes. Merge each one:
  ```bash
  git status                                  # clean: commit your work first
  git fetch upstream --tags
  git tag -l 'v*' --sort=-v:refname | head -3
  git merge v0.10.19
  npm install && npm run db:migrate && npm run check
  ```
  Git merges line by line: a file only the template changed takes its version, a file only you
  changed keeps yours, and a place you both changed is a conflict to solve:
  - `package-lock.json`: `git checkout --theirs package-lock.json`, then `npm install`, then commit.
  - The template's files (`src/platform/`, `docs/agent/`, `scripts/platform.mjs`, `CHANGELOG.md`,
    `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `DEVELOPERS.md`, `LICENSE`): `git checkout --theirs <file>`.
  - Your files: keep your work, bring over what the template changed.
  - `git merge --abort` puts everything back.

  Then follow the release's `docs/agent/upgrade.md`, download the kit again for its new
  `store-check.mjs`, run it, push, and submit an update.
- **Your app's updates** (to customers): every new version you submit goes through review; once
  approved, customers see "Update available", and it is merged into their copy (their changes
  stay; clashes go to their AI). Keep updates small, don't move files without reason, and only add
  migrations. [Updates](https://ezystudio.com/developers/docs/updates).

### From a zip

A zip download has no history in common with this repository. Connect it once, then merge releases
as above:

```bash
git remote add upstream https://github.com/EzyStudio/web-app-template.git
git fetch upstream --tags
git merge v<the version in your template.json> --allow-unrelated-histories   # once
```

That first merge lists the files that differ from the release (usually only the ones you changed):
keep your side of each (`git checkout --ours <file>`), then commit.

## 10. Troubleshooting

`README.md` › Troubleshooting covers setup (Node version, port 5432 taken, database not running,
503 `auth_not_configured` without `JWT_SECRET`, pending migrations). For the store:

| What you see | Fix |
| --- | --- |
| Submit says `template_outdated` | merge the newest release ([How updates work](#9-how-updates-work)), then download the kit again |
| Submit says `platform_changed` or `instructions_changed` | `git checkout upstream/main -- src/platform AGENTS.md CLAUDE.md GEMINI.md docs/agent` (or the tag you are on), then commit |
| `store-check.mjs` fails on `.claude/` or `.vscode/` | `git rm -r --cached .claude .vscode` and add them to `.gitignore` |
| `store-check.mjs` says the kit is older than your template | download the kit again |

## 11. FAQ

**Do I need a Site Key or an EzyStudio account to build?** No: everything runs locally. You need
an approved developer account only to submit.

**Can I test the owner's "View as admin"?** Not locally. On your public preview, "Get admin login"
gives your team admin accounts.

**Where do my own docs go?** `README.md`. Never a second `AGENTS.md` or another AI instruction file:
submit refuses them.

**Can I host the app elsewhere?** No: the license allows building sites and apps that run on
EzyStudio.

## 12. For your AI

Exact steps for a coding agent with only this repository. Run each command from the repository
root; stop and report if one fails.

1. **Every task:** read `AGENTS.md`, then the `docs/agent/` topic for the work. This file covers
   setup and updates only.
2. **Set up** (once):
   1. `node -v` must be 22.12 or later; otherwise tell the developer to install it (`.nvmrc`).
   2. `npm install`
   3. `docker compose up -d --wait db` (if port 5432 is taken: add `DB_PORT=5433` to `.env` and
      run it again).
   4. `npm run setup` — it writes `.env` (a new `JWT_SECRET`, the local `DATABASE_URL`, an empty
      `SITE_KEY`) and runs the migrations. Never put a real `SITE_KEY` in `.env`.
   5. Optional: `npm run db:seed`.
3. **Run it and create an admin:**
   1. `npm run dev`. Open http://localhost:4321 (or the address it prints).
   2. Sign up at `/account/register/` with the developer's address (in the browser, or ask the
      developer to).
   3. The 6-digit code is in the dev server's output under "LOCAL DEVELOPMENT ONLY"
      (`npx astro dev logs` if it runs in the background). Type it on the verify page.
   4. `npm run dev-admin -- <that address>`; reload `/dashboard/`.
   Never give anyone `owner`, never write roles into the database yourself, never set `SITE_KEY`
   or `NODE_ENV=production` to get around a refusal.
4. **Update** when the developer asks, or `template.json` is below the newest release:
   1. `git status` must be clean (commit first).
   2. `git fetch upstream --tags`; newest: `git tag -l 'v*' --sort=-v:refname | head -1`.
   3. `git merge <tag>`. Resolve: `package-lock.json` → `git checkout --theirs` then `npm install`;
      the template's files listed in [How updates work](#9-how-updates-work) → `--theirs`; the
      developer's files → keep theirs and bring over the template's change.
   4. `npm install`, `npm run db:migrate`, `npm run check`, then the new `docs/agent/upgrade.md`
      from step 2.
   5. If it can't finish: `git merge --abort`, and say so.
   6. Tell the developer what changed, and to download the kit again and run `node store-check.mjs`.
5. **Before saying a change is done:** `npm run check`, and `node store-check.mjs` when it is there.

## 13. Reporting problems and suggesting changes

This repository is where releases are published; it is read-only for everyone outside the team.

- **Nobody pushes to `main`.** `main` and the `v*` release tags are protected: they change only
  with a release. Work in your own repository (`origin`), never in this one.
- **Found a bug, or something unclear?** Open an issue here, with the template version
  (`template.json`), what you did and what you saw. Never paste a `.env`, a key or customer data.
- **Want to suggest a change?** Fork this repository and open a pull request against `main`. We
  review it, and when we take it, the change goes out in the next release (the pull request is then
  closed with "Included in vX.Y.Z"). Pull requests are not merged here directly, because every
  release replaces these files from our own source.
- **Security problem?** Don't open a public issue: email support@ezystudio.com with "Security" in the subject.
