---
name: job-dashboard-sync
description: >-
  Reconcile the Job Match Analysis dashboard artifact against Job Applications Tracker.xlsx and the Resume Tailoring folder structure. Use whenever [Your Name] asks "did you already run this", "why isn't this showing up", "is my dashboard up to date", before/after any resume pipeline run, or any time an application status changes. Catches stale "pending" states, missing cards, stale job URLs, and out-of-sync Applied badges.
---

# Job Dashboard Sync Check

Three sources of truth exist for your job search and they drift apart over time:

1. **Filesystem** -- Resume Tailoring/<Company>/<Role>/ folders (a resume .docx inside means the role has been processed)
2. **Job Applications Tracker.xlsx** -- the source of truth for application status (Applied, Phone Screen, Interview, etc.)
3. **job-match-analysis artifact** -- the dashboard you actually look at

Never trust the artifact's existing data on its own. Always reconcile against the other two.

## Steps

1. **Get ground truth from the filesystem** -- list all subfolders under your Resume Tailoring directory that contain a resume .docx. Each of these is a processed role. Use bash:

```bash
find "[RESUME_TAILORING_PATH]" -maxdepth 3 -name "*.docx" | grep -i "Resume -" | sed 's|/[^/]*$||' | sort -u
```

2. **Read the tracker** -- open Job Applications Tracker.xlsx with openpyxl and extract every row on the Applications tab: Company, Role, Status, Source, Job URL, Resume File, Notes.

3. **Get the live artifact** -- use mcp__cowork__list_artifacts, find job-match-analysis, and Read the file to get the current ROLES array.

4. **Diff ground truth against the artifact ROLES array**, matching by company + role (fuzzy match -- tracker and artifact wording can differ slightly). For each ground-truth record:
   - **No matching card exists** -> add one. If the original score/dimensions aren't known, do NOT invent them -- use score: "--", verdict: "Unscored", and note in cons that the card was added via sync check and needs re-scoring.
   - **Card exists but state is "pending" while the folder has a resume .docx** -> fix to "processed", add resumeFile.
   - **Card's applied/appliedDate doesn't match the tracker** -> update it.
   - **Card's jdUrl doesn't match the tracker's Job URL** -> update it. Re-verify the req ID is still live before assuming the tracker is right -- companies rotate req IDs on reposts.

5. **Only write back if you found at least one mismatch.** Call mcp__cowork__update_artifact with id: "job-match-analysis" and the updated HTML. Summarize exactly what changed.

6. **Report** in one short paragraph: what was out of sync and what you fixed. If nothing was out of sync, say so in one sentence.

## Status -> applied mapping

**External roles:**
- Applied, Phone Screen, Interview, Offer, Rejected, Withdrawn -> applied: true
- Not Applied -> applied: false

**Internal roles:**
- Info Interview Requested, Info Interview Scheduled, First Round Interview, Offer, Not Interested, Withdrawn -> applied: true (treat "in motion" as applied for internal roles, since there's no formal external application step)
- Identified -> applied: false

## Notes

- Internal roles don't always get cards on the Job Match Analysis -- only if a resume was actually generated. Check before adding one.
- This skill only reconciles data. It does not score new roles (use the job-score skill) and does not run the resume pipeline.
