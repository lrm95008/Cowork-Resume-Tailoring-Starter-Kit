# Resume Tailoring Assistant — What Is This?

This kit sets up an automated job-search pipeline inside Claude. Once it's running, you share a job lead — a URL, a pasted description, a forwarded email — and Claude handles the rest: finding the real job posting, tailoring your resume, drafting a cover letter or introductory email, filing everything, and logging it in a tracker. You end up with submission-ready documents in seconds instead of hours.

---

## The Problem It Solves

Applying to multiple roles means doing the same work over and over: rewriting your resume for each job description, keeping track of where you've applied, figuring out which leads are worth your time, and making sure nothing falls through the cracks. This pipeline automates the repetitive parts so you spend your time on interviews, not formatting.

---

## What Happens When You Share a Job Lead

Before the pipeline runs, the role is scored 1–5 across five dimensions — salary fit, level match, scope match, technical fit, and domain fit — and given one of three verdicts:

- **Process (4.0–5.0):** Strong fit. The seven-step pipeline runs automatically with no confirmation needed.
- **Review (3.0–3.9):** Possible fit with concerns. Claude surfaces the gaps and asks whether to proceed — you decide.
- **Skip (below 3.0):** Poor fit. Claude explains why and stops. No resume work happens.

For Process verdicts, the pipeline continues through seven steps automatically:

**1. Gets the real job description.**
If you paste a LinkedIn or job-board link, it finds the same posting on the company's own careers page — the canonical version recruiters and ATS systems use.

**2. Creates a folder.**
A dedicated folder is created for that company and role to hold every file associated with that application.

**3. Saves the job description as a PDF.**
The full posting is saved for reference, so you always have the original even if the company removes it later.

**4. Tailors your resume.**
The top 10–15 keywords and priorities from the job description are identified, then your resume is rewritten and reordered to surface the most relevant experience first. Nothing is invented — only what's already true about you, repositioned.

**5. Drafts a companion document.**
For external roles, a tailored cover letter. For internal roles at your current employer, a short introductory email requesting an informational interview. Both are generated automatically alongside the resume — you don't have to ask.

**6. Logs everything in your tracker.**
The role, status, resume file path, and job URL are added to your Job Applications Tracker spreadsheet. When you report progress — a phone screen, an interview, an offer — Claude updates the tracker for you.

**7. Syncs the dashboards.**
The Job Match Analysis board is updated so scored roles never show as stale or missing, and the Status Board reflects the latest application state.

---

## The Three Dashboards

Alongside the pipeline, three live dashboards keep the whole search visible at a glance.

### Job Match Analysis
A scoring board — one card per role you've evaluated. Each card shows the five dimension scores, the overall score, the verdict, salary, location, and an Applied badge once you've submitted. Filtered by verdict or pipeline state. Updated automatically every time the pipeline processes a new role.

Best for: deciding which roles to prioritize and seeing your search at a strategic level.

### Job Hunt Status Board
An application status board — all your active roles as cards, color-coded by where they sit in the process (Not Applied → Applied → Phone Screen → Interview → Offer). Split into External and Internal sections with summary stats at the top. A Refresh button re-reads your tracker and updates the board in place.

Best for: daily check-ins — what's active, what needs a follow-up, what's stalled.

### Gmail Activity Feed
A live email feed that searches your inbox every time you open it and classifies what it finds — direct outreach from companies, internal job notifications, LinkedIn recruiter messages, and job alerts. No manual updating — it's always current because it reads from Gmail directly.

Best for: making sure no new lead goes unnoticed and having a quick view of incoming recruiter activity.

---

## What Your Workflow Looks Like Once It's Running

**When a new role comes in:** Paste the URL into a conversation. The pipeline runs. Two minutes later you have a tailored resume, a cover letter, a filed PDF, and a tracker entry — ready to submit or set aside.

**In the morning:** Open the Gmail Activity feed to see what came in overnight. Check the Status Board for anything that needs a follow-up. Glance at the Job Match Analysis to see where the strongest opportunities sit.

**When something happens:** Tell Claude — *"I have a phone screen with Google Thursday"* — and it updates the tracker and both status dashboards in the same response.

**When you want to verify everything is in sync:** Say *"sync my dashboards"* and Claude cross-checks the artifacts against your tracker and files, fixing anything that's drifted.

---

## What's in This Kit

| File | What it's for |
|------|--------------|
| `Resume Tailoring Assistant - Overview.pptx` | Slide deck overview — useful for explaining the system to others |
| `CLAUDE.md` | The pipeline instructions. Add this to your Claude project and fill in the bracketed placeholders with your own details. |
| `Setup Tutorial.md` | Step-by-step walkthrough — from creating your Cowork project through getting all three dashboards running. Start here once you've read this overview. |
| `Job Applications Tracker.xlsx` | The tracker template. Same columns the pipeline expects, pre-formatted. |
| `skills/job-score.md` | Custom skill template — auto-scores any role you share and adds a card to your Job Match Analysis. Fill in your profile placeholders before creating the skill. |
| `skills/status-update.md` | Custom skill template — auto-updates your tracker and Status Board when you report application progress. |
| `skills/dashboard-sync.md` | Custom skill template — auto-reconciles your Job Match Analysis when you ask about pipeline state. |

---

## Where to Start

Read this overview (done). Then open `Setup Tutorial.md` and follow it from Stage 1. The whole setup takes about 20–30 minutes the first time; after that the pipeline runs itself.
