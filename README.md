# How to Use the `ai-job-search` Framework

A detailed, practical walkthrough — tailored to targeting **DevOps / Cloud / intern roles in France**.

Repo: <https://github.com/MadsLorentzen/ai-job-search>

---

## What it actually is

It's **not an app you run** — it's a **template GitHub repository** that turns Claude Code (the CLI) into a job-application assistant. You clone it, fill it with *your* data, then drive it with slash commands (`/setup`, `/scrape`, `/apply`, …). All the "AI" is Claude Code reading the repo's instruction files and acting on them. Nothing runs on someone else's server; your CV and personal data stay on your machine.

Think of it as: **your profile (files) + job postings (fetched) → tailored CV + cover letter (PDFs) + interview prep.**

---

## Important reality check for the France case

1. **The built-in job portals are Danish** (Jobindex, Jobnet, etc.) — not useful for France. But it ships **two country-agnostic ones**:
   - `linkedin-search` — hits LinkedIn's *public guest* job endpoints (no login, no API key). Works for France via `-l "Paris, France"`. This is the compliant, lightweight alternative to a paid scraper (e.g. Composio/Apify) — but keep volume low (automated access is against LinkedIn ToS).
   - `freehire-search` — a tech-job aggregator (software / data / DevOps), multi-market via `--country`. **Well-suited to DevOps/Cloud goals.**
2. For proper French coverage, add local boards (APEC, Welcome to the Jungle, HelloWork, Indeed FR) using `/add-portal`.

---

## Prerequisites to install first

| Requirement | Why | Note |
|---|---|---|
| **Claude Code** | The engine | Needs a paid Claude plan or API credits. |
| **Python 3.10+** | ATS checks, tools | Likely already installed on Linux. |
| **Bun** | Runs the job-search CLIs | `curl -fsSL https://bun.sh/install \| bash` |
| **LaTeX** (lualatex + xelatex) | Compiles CV & cover letter to PDF | Ubuntu: `sudo apt install texlive-full` (large) or TinyTeX + extra packages. |
| **git / gh** | Clone / version control | |
| Optional: `pip install pypdf` | ATS parseability check | Recommended. |

---

## Step-by-step usage

### 1. Clone — use a PRIVATE repo (do NOT fork)
⚠️ **Critical:** `/setup` writes your real name, contact info, employment history, and salary expectations into tracked files. A GitHub *fork* of a public repo **is always public** — your CV data would be exposed. For a personal job search (not contributing code back), do **not** fork. Instead:

```bash
git clone https://github.com/MadsLorentzen/ai-job-search.git
cd ai-job-search
# then create your OWN private repo and point this at it as upstream
```
(Exact recipe is in the repo's `SETUP.md`, section 8. Fork only if you plan to contribute improvements upstream.)

### 2. Install the job-search tools
```bash
for tool in linkedin-search freehire-search; do
  (cd .agents/skills/$tool/cli && bun install)
done
```
(Skip the Danish ones. `linkedin-search` and `freehire-search` run with zero runtime dependencies.)

### 3. Set up your profile
```bash
claude          # launch Claude Code inside the repo folder
/setup
```
`/setup` offers three paths (auto-detected):
- **Documents folder** — drop your CV PDF, LinkedIn export, diplomas into `documents/` and it reads them (best, richest).
- **Paste a CV** in chat.
- **Interview** — it asks you questions.

Produces your profile files (`CLAUDE.md`, `01-candidate-profile.md`, etc.). **Profile depth = output quality.** Describe *what you did* (projects, tools, results), not just titles — e.g. "Built CI/CD pipelines with GitLab CI + Terraform on AWS EKS" beats "DevOps".

It also captures **languages** — important for France: it hard-rejects postings requiring a language you didn't declare, so record French/English levels correctly.

### 4. Configure your search (roles / locations)
```bash
/setup --section search
```
Set: target roles = *DevOps Engineer, Cloud Engineer, DevOps/Cloud intern*; location = *France / Paris / Remote*; which portals to use.

### 5. Add French job boards (recommended)
```bash
/add-portal
# e.g. https://www.welcometothejungle.com  or  https://www.apec.fr
```
It investigates the board, generates a search skill, checks robots.txt/ToS, and test-runs before registering. Repeat per board.

### 6. Scrape jobs
```bash
/scrape
```
Searches all enabled portals, deduplicates, presents matches sorted by fit. If many results:
```bash
/rank
```
Batch-scores every posting against the fit framework → ranked shortlist with per-job strengths/gaps (deal-breakers veto, deadlines flagged).

### 7. Apply to a specific job
```bash
/apply https://www.welcometothejungle.com/fr/companies/x/jobs/devops-engineer_paris
# or, if the site blocks fetching, paste the full job text:
/apply <paste job description>
```
The full pipeline:
1. Evaluate fit against your profile
2. Draft a tailored CV + cover letter in LaTeX
3. A **second Claude agent** researches the company and critiques the drafts
4. Revise
5. **Compile & visually inspect PDFs** — iterates until CV is exactly 2 pages (no orphaned titles) and cover letter is 1 page
6. **ATS-check** — extracts the PDF text layer and verifies a parser sees your contact info and keywords correctly (never fabricates skills — genuine gaps stay visible)
7. Presents final PDFs + a verification checklist

---

## Extra commands (once your profile exists)

- `/interview` — stage-specific prep pack + mock interview for a tracked application
- `/outcome` — record results (interview/offer/rejection), archive materials; `/outcome followup` drafts polite nudges for quiet applications
- `/upskill` — analyzes gaps between your profile and target jobs, produces a learning plan
- `/expand` — enriches your profile from your public GitHub/portfolio/etc.
- `/html-report` — offline dashboard of your application pipeline
- `/notion-sync`, `/gmail-sync` — mirror pipeline to Notion / detect status from Gmail
- `/add-template` — use your own CV/cover-letter design instead of the stock LaTeX one
- `/reset` — wipe profile data to start over

---

## Suggested path

1. Install Bun + LaTeX + pypdf
2. Clone as a **private** repo (not fork)
3. `bun install` in `linkedin-search` and `freehire-search`
4. Put your CV in `documents/cv/`, run `/setup`
5. `/setup --section search` → DevOps/Cloud/intern, France
6. `/add-portal` for Welcome to the Jungle + APEC
7. `/scrape` → `/rank` → `/apply` on the best matches

---

## Notes / caveats

- The slash commands (`/setup`, `/scrape`, `/apply`, …) are run **inside the framework's own Claude Code session** — you type them yourself.
- `linkedin-search` is for **personal, low-volume** use only (LinkedIn ToS).
- Job postings are treated as **untrusted input**; still skim what was fetched/written before sending on unfamiliar boards.
- This is an independent open-source project, **not affiliated with Anthropic**.
