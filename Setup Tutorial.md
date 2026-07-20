# Complete Setup Guide — Resume Tailoring Pipeline & Dashboards

This is the end-to-end walkthrough. Follow it in order. Each section builds on the previous one — the pipeline has to be working before the dashboards make sense.

---

## Stage 1 — Create Your Cowork Project

Everything lives inside a single Cowork project. This is what scopes Claude's behavior to your job search — instructions, files, and memory all stay contained here.

**1.1** Open Claude for Desktop and switch to Cowork mode.

**1.2** Create a new project. Name it something like *Resume Tailoring* or *Job Search.*

**1.3** Connect a folder on your computer to this project. This is where your files will live — resumes, job descriptions, tracker. Pick a dedicated folder, not your Downloads or Desktop.

---

## Stage 2 — Add the Pipeline Instructions

The `CLAUDE.md` file is the brain of the pipeline. It tells Claude what to do every time you share a job lead, without you having to repeat yourself.

**2.1** Copy the `CLAUDE.md` file from this kit into the folder you connected in Stage 1.

**2.2** Open `CLAUDE.md` and replace every `[bracketed placeholder]` with your own details. Work through them in order:

- **`[Your Name]`** — appears in resume filenames and the resume header. Use whatever name you put on your resume.
- **Target job titles** — in the *Target Roles & Salary Threshold* section, replace the example role names with the actual titles you're hunting for. Be specific — "Principal Product Manager" and "Director of Engineering" are better than vague categories.
- **`[Your Target Salary Threshold]`** — the minimum total comp top-of-range you'd accept for conditional roles (e.g., Senior-level titles). Below this number, Senior roles don't get auto-processed — they get surfaced for your decision instead.
- **Your profile** — in the *Job Scoring* section, fill in your current role, years/domain of experience, top strengths, and location preference. This is what every lead gets scored against.
- **Formatting rules** — the defaults are a good starting point. Change font, point sizes, or margins only if your existing resume style differs significantly.
- **Internal job pipeline section** — if your employer posts internal roles via email notifications, keep this section and fill in the notification sender address. If not, delete it.

**2.3** Save `CLAUDE.md` in your project folder. Claude reads it automatically at the start of every conversation in this project.

---

## Stage 2.5 — Install Skills

The pipeline relies on skills for document generation and workflow automation. Some are built into Cowork; three are custom skills included in this kit that you create once.

**Built-in skills to install:**
In Cowork, go to **Settings → Skills → Browse** and install:
- `docx` — generates Word documents (resumes, cover letters, introductory emails)
- `pdf` — saves job descriptions as PDFs
- `xlsx` — reads and updates your tracker spreadsheet
- `schedule` — creates recurring scheduled tasks

**Custom skills to create (templates in the `skills/` folder of this kit):**
These three skills add reliable auto-triggering on top of your CLAUDE.md — without them Claude will still do the work, but less consistently:

- `job-score.md` — auto-scores any role you share and adds a card to your Job Match Analysis
- `status-update.md` — auto-updates your tracker and Status Board when you report progress
- `dashboard-sync.md` — auto-reconciles your Job Match Analysis when you ask about pipeline state

**How to create each custom skill:**
In a conversation in your project, say:
> *"I'd like to create a new skill. Here's the content:"*

Then paste the full contents of the skill template file. Claude will use the skill-creator to build and save it. Do this for all three skill files before moving on.

**Fill in the placeholders first:** Each template has `[bracketed placeholders]` for your name, target roles, salary threshold, and tracker file path. Fill these in before pasting — the skill works from this content each time it's triggered.

---

## Stage 3 — Add Your Base Resume

**3.1** Save your current resume as a `.docx` file named `Base Resume - [Your Name].docx` in your project folder. This is the master copy — the pipeline always copies from it, never edits it directly.

**3.2** Open a conversation in your project and say:
> *"Can you read my base resume and confirm you can see it?"*

Claude should respond with a brief summary of your resume's structure. If it can't find the file, double-check the filename matches exactly what you put in `CLAUDE.md`.

---

## Stage 4 — Set Up the Tracker

**4.1** Copy `Job Applications Tracker.xlsx` from this kit into your project folder.

**4.2** Delete the three placeholder rows — they're there to show you the column structure, not to keep.

**4.3** Confirm the columns are correct: Company, Source, Role, Date Added, Status, Job URL, Resume File, Notes, Position ID. Don't rename or reorder these — the pipeline references them by name.

---

## Stage 5 — Test the Pipeline with One Job Lead

Before building any dashboards, verify the pipeline runs end-to-end on a real role. This catches any setup issues before you have a dozen roles in the system.

**5.1** Find one job posting you're genuinely interested in. Copy the URL.

**5.2** Paste it into a conversation in your project with a message like:
> *"Here's a role I want to look at: [URL]"*

Claude will first score the role against your profile (1–5 across five dimensions):

- **4.0–5.0 → Process**: pipeline runs automatically, no confirmation needed
- **3.0–3.9 → Review**: Claude surfaces the key concerns and asks whether to proceed — you decide
- **Below 3.0 → Skip**: pipeline stops; Claude explains the gaps

If the verdict is Process (or Review and you say go), it runs seven steps:

- **Step 1:** Fetch the job description from the company's own careers page (not the job board)
- **Step 2:** Create a folder for that company and role
- **Step 3:** Save the job description as a PDF
- **Step 4:** Tailor your resume against the top keywords and priorities from the JD
- **Step 5:** Generate a cover letter (external roles) or introductory email (internal roles)
- **Step 6:** Log the role in your tracker with status "Not Applied" (or "Identified" for internal roles)
- **Step 7:** Update the Job Match Analysis dashboard so the role appears with its score and state

**5.3** Check the output. You should see a folder created inside your project folder with the company name, a PDF of the job description, a tailored `.docx` resume, and a cover letter or email `.docx`. Your tracker should have a new row.

**5.4** If anything is missing or wrong, tell Claude what you expected and it will fix it. Common first-run issues:
- *Base resume not found* → check the filename in `CLAUDE.md` matches exactly
- *Wrong job description fetched* → the company's careers page may need a direct URL; paste it and try again
- *Tracker not updated* → confirm `Job Applications Tracker.xlsx` is in the root of your project folder, not a subfolder

Don't move to Stage 6 until one full pipeline run completes cleanly.

> **Two ways to trigger the pipeline.** What you just tested is the ad-hoc trigger — you share a lead, the pipeline runs immediately. There's also a scheduled trigger: a daily Gmail scan that runs automatically every morning, surfaces new matching leads from your inbox, and feeds them into the same pipeline without you having to share anything. Setting that up is covered in Stage 9 alongside the Gmail Activity dashboard.

---

## Stage 6 — Connect Gmail (Required for automated scanning and Gmail Activity)

The Gmail connector enables two things: the **daily job scan** (which automatically finds leads in your inbox every morning and runs them through the full pipeline into the Job Match Analysis) and the **Gmail Activity Feed artifact** (the live email view you'll create in Stage 9). Both require this connector. If you plan to share leads manually only and don't want either feature, skip to Stage 7 — but you'll miss the automated daily scanning.

**6.1** In Cowork, go to **Settings → Connectors**.

**6.2** Add the Gmail connector and authorize it with the Google account where you receive job-related emails.

**6.3** Test it by asking Claude:
> *"Can you search my Gmail for any emails mentioning 'interview' or 'opportunity' in the last 30 days?"*

If Claude can return results, the connector is working. If you get an authorization error, re-authorize in Settings → Connectors.

---

## Stage 7 — Create the Job Match Analysis Artifact

Now that the pipeline is running, the first artifact to create is the scoring board. You need at least one scored role in your tracker before this is useful.

**What it is:** A dark-mode card grid — one card per role you've evaluated, showing scores, verdict, salary, and an Applied badge. Filtered by verdict or state. Updated automatically by the pipeline whenever a new role is processed.

**How to create it:** In a conversation in your project, say:
> *"Please create a Job Match Analysis artifact for me. It should be a dark-mode card grid showing every role I've scored so far, with each card showing: company, role title, location, salary range, scores across the five dimensions (salary fit, level match, scope match, technical fit, domain fit — each out of 5), an overall score, a Process/Review/Skip verdict badge, and an Applied/Not Applied badge with date. Add a filter bar to filter by verdict and state. Populate it from the roles currently in my tracker."*

Claude will generate the HTML and save it as an artifact called `job-match-analysis`. It will appear in your artifact panel in Cowork.

**How it stays current:** Every time the pipeline processes a new role, Step 7 in your `CLAUDE.md` instructs Claude to update this artifact automatically — adding the new card and setting its state. You don't need to ask.

When you want to verify it's accurate, say: *"Please sync my Job Match Analysis artifact against my tracker."*

---

## Stage 8 — Create the Job Hunt Status Board

**What it is:** A light-mode application status board — all your active roles as cards, color-coded by where they sit in the process, split into External and Internal sections. Summary stats at the top. A Refresh button that re-reads your tracker and rewrites the board in place.

**How to create it:** Say:
> *"Please create a Job Hunt Status Board artifact for me. It should be a light-mode dashboard showing all my applications as cards, split into External and Internal sections. Each card should show the company (with a colored initial logo), role title linked to the job URL, a status badge color-coded by stage, date added, and notes. Include summary stat tiles at the top. Add a Refresh button that sends a prompt asking you to re-read my tracker and update the board. Populate it from my current tracker data."*

**Status badge color guide to give Claude:**

| Status | Color |
|--------|-------|
| Offer | Green |
| Interview / First Round Interview | Blue |
| Phone Screen / Info Interview Scheduled | Teal |
| Info Interview Requested / Applied | Amber |
| Not Applied / Identified | Grey |
| Rejected / Withdrawn / Not Interested | Red/muted |

**How it stays current:** The Refresh button in the artifact sends a pre-written message to Claude asking it to re-read your tracker and rewrite the embedded data. You can also just tell Claude *"I just got a phone screen with [Company] — update my status board"* and the pipeline handles it directly.

---

## Stage 9 — Create the Job Hunt Gmail Activity Feed

**What it is:** A live email thread list — searches your Gmail inbox every time you open it and classifies what it finds into categories (External outreach, Internal notifications, Recruiter, Job Alert). Never needs manual updating — it's always current.

**Prerequisite:** Gmail connector must be authorized (Stage 6).

**How to create it:** Say:
> *"Please create a Job Hunt Gmail Activity artifact for me. It should search my Gmail inbox every time it's opened using the Gmail connector and display recent job-related threads as a list. Classify each thread as: Internal (from my employer's job notification address), External (from a company's own domain), Job Alert (from LinkedIn), Recruiter (LinkedIn InMail or similar), or Other. Add filter buttons for each category. Use this search query:"*

Then paste this query (customizing the sender addresses for your own situation):

```
(subject:(interview OR offer OR application OR "thank you for your interest" OR "next steps" OR informational OR opportunity OR "moving forward")
OR from:[your-employer-notification-address]
OR from:jobs-listings@linkedin.com
OR from:inmail-hit-reply@linkedin.com)
newer_than:30d
```

**How to customize the classifier:** Tell Claude which sender addresses belong to which category. For example, if your employer sends internal job notifications from `jobs@yourcompany.com`, tell Claude to classify that sender as "Internal." The more specific you are here, the cleaner the categorization.

**9.4 — Schedule the daily Gmail job scan**

Now that the Gmail connector is authorized and the pipeline is tested, schedule the scan to run automatically every morning so you never have to trigger it manually.

In a conversation in your project, say:
> *"Please create a scheduled task that runs every weekday morning at 8am. It should search my Gmail inbox for new job leads matching my target roles, score any new results, and run the full pipeline automatically on any role that scores 4.0 or higher."*

Claude will create a recurring task using Cowork's scheduler. To confirm it's set up, ask: *"What scheduled tasks do I have?"* — Claude will list all active tasks and their next run times.

To adjust the time or cadence later: *"Change my daily job scan to run at 7am instead."*

**9.5 — Schedule the weekly stale application check**

The second scheduled task watches for applications that have gone quiet. It scans your tracker for any row where Status = "Applied" and the date is 14 or more days in the past, then surfaces a follow-up reminder so nothing goes cold without being noticed.

In a conversation in your project, say:
> *"Please create a weekly scheduled task that runs every Monday morning. It should read my Job Applications Tracker, find every row where Status is 'Applied' and the date is 14 or more days before today, and present a follow-up reminder listing the company, role, date applied, and days since — or confirm that all active applications are recent or have already progressed."*

The task reads your tracker at runtime (no hardcoded dates) and skips rows that are Not Applied, already past the Applied stage (Phone Screen, Interview, Offer, etc.), or have no date. Results appear directly in chat — no file output.

To run it manually any time: *"Run my stale application check now."*

Once both tasks are created, your pipeline is fully automated: new leads are found and processed every morning, and any application that goes quiet for two weeks gets flagged before it slips through the cracks.

---

## Stage 10 — Verify Everything Works Together

With all three artifacts created and the pipeline running, do one more end-to-end test.

**10.1** Share a second job lead and watch the pipeline run. When it finishes, open your Job Match Analysis — the new card should be there.

**10.2** Tell Claude you've applied to one of the roles. Check that the tracker row updates, the Status Board reflects the new status, and the Applied badge on the Job Match Analysis card flips to green.

**10.3** Open the Gmail Activity feed and confirm it's pulling threads from your inbox.

**10.4** Ask Claude: *"Can you sync all my dashboards against my tracker?"* This runs the Step 7a reconciliation check from `CLAUDE.md` — it cross-checks your artifact data against your tracker file and actual folder structure, and fixes anything that's drifted. Run this any time things look out of sync.

---

## Troubleshooting

**Pipeline ran but no folder was created:**
Confirm your project folder is properly connected in Cowork settings. If it is, try asking Claude: *"Where are you saving files? Can you tell me the path you're using?"*

**Job Match Analysis shows "pending" cards for roles that were already processed:**
Run the sync check: *"Please sync my Job Match Analysis against my tracker and folders."* Pending cards that have a corresponding folder and resume file will be flipped to processed.

**Status Board doesn't update after I click Refresh:**
The Refresh button sends Claude a message, which Claude must then process. If nothing happens, start a new conversation in your project and say: *"Please refresh my Job Hunt Status Board from my tracker."*

**Gmail Activity shows an error or blank screen:**
Re-check your Gmail connector in Settings → Connectors. If it's authorized but still failing, try re-opening the artifact — the Gmail API occasionally times out on the first call and succeeds on retry.

**A role on the Job Match Analysis has the wrong job URL:**
Companies (especially large tech) rotate their job req IDs when they repost a role. Say: *"The job URL for [Company] [Role] on my Job Match Analysis is stale — can you find the current one and update the card?"*

**The pipeline auto-processed a role I didn't want to pursue:**
Add a "Skip without asking" rule to your `CLAUDE.md` for that role type, company, or domain pattern so it doesn't happen again.
