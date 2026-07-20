# Resume Tailoring Pipeline

## File Layout
```
Resume Tailoring/
  Base Resume - [Your Name].docx     ← MASTER — never modify, always copy
  Job Applications Tracker.xlsx      ← one row per job, updated by pipeline
  pipeline_helper.py                 ← utility script for folder/PDF/tracker ops
  <Company>/
    <Role>/
      Job Description - <Company> - <Role>.pdf
      [Your Name] Resume - <Company> - <Role>.docx
      Cover Letter - <Company> - <Role>.docx              ← External roles only
      Introductory Email - <Company> - <Role>.docx         ← Internal roles only
```

## Full Pipeline — Triggered Any Time a Job Is Provided

When the user shares a job URL, pastes a job description, or forwards a job lead,
run every step below automatically without asking for confirmation.

### Step 1 — Get the Real Job Description
- If a URL is provided, fetch the company's own careers page (NOT LinkedIn/Indeed).
  If the URL is a job board, extract the company name and role title,
  then search for the same posting on the company's website.
- Extract: company name, role title, full job description text, job URL.

### Step 2 — Create the Folder
```bash
cd "$(dirname "$0")"   # Resume Tailoring/
python pipeline_helper.py create-folder "<Company>" "<Role>"
```
Returns the folder path, e.g. `Resume Tailoring/Google/Senior PM/`.

### Step 3 — Save the Job Description as PDF
Write the full job description text to a temp file, then:
```bash
python pipeline_helper.py save-pdf "<Company>" "<Role>" /tmp/jd_text.txt
```
Output: `<Company>/<Role>/Job Description - <Company> - <Role>.pdf`

### Step 4 — Tailor the Resume
- Read `Base Resume - [Your Name].docx` (the master).
- Identify the top 10-15 keywords, skills, and priorities from the job description.
- Rewrite/reorder bullets to surface the most relevant experience first.
- Adjust the headline (section after the name) to mirror the role language.
- Keep all experience true and in your own voice — do NOT invent anything.
- Max two pages, consistent font throughout (see formatting rules below).

### Step 5 — Save the Tailored Resume
Use the docx skill to produce:
`<Company>/<Role>/[Your Name] Resume - <Company> - <Role>.docx`

### Step 5a — Generate the Companion Document
Run automatically for every role, no need to ask:

- **External roles (Source = External):** Generate a tailored cover letter.
  - Save as `<Company>/<Role>/Cover Letter - <Company> - <Role>.docx`
  - Same formatting rules as the resume (one page max, left-justified, no horizontal rules).
  - Professional tone, references 2-3 of the most relevant achievements from the tailored
    resume, ties them directly to the job description's top priorities. Do not invent
    anything not already in the resume.
  - Standard business letter structure: contact block, date, salutation, 3-4 short
    paragraphs, sign-off with your name.

- **Internal roles (Source = Internal):** Generate a short introductory email instead
  of a cover letter.
  - Save as `<Company>/<Role>/Introductory Email - <Company> - <Role>.docx`
  - Short (3-5 sentences): states you're requesting an informational interview for the role,
    mentions the attached resume, and expresses interest tied to 1-2 specifics from the JD.
  - Format as a plain email (Subject line, greeting, body, sign-off) — not resume
    formatting/fonts. Use a placeholder like "[Hiring Manager Name]" for the
    recipient unless you've given a name.
  - This is a draft for you to send yourself — do not send it automatically unless you
    explicitly ask.

### Step 6 — Update the Tracker
```bash
python pipeline_helper.py update-tracker \
  "<Company>" "<Role>" "<YYYY-MM-DD>" \
  "<Company>/<Role>/[Your Name] Resume - <Company> - <Role>.docx" \
  "<job_url>" "Not Applied"
```

### Step 7 — Update the Job Match Analysis Artifact (optional)
If you maintain a live dashboard artifact for scored roles, update it here:
- Change the role's `state` from `"pending"` to `"processed"`
- Set the correct `jdUrl` (company careers page, not a job board — re-verify the req ID is
  still live; companies rotate req IDs on reposts, so an old link can look "missing" even
  though it's the same role)
- Add `resumeFile` pointing to the .docx just created
- Set `applied: false` and `appliedDate: null` (Step 6 just wrote "Not Applied"/"Identified" to the tracker)
- Add the role as a new entry if it doesn't exist yet

This step is optional — skip it if you're not using a dashboard artifact, or adapt it to
whatever status board / tracking tool you use instead.

### Step 7a — Dashboard Sync Check (recommended every pipeline run, and whenever you ask about status)
Before ending any pipeline run, and any time you ask "did you already run this," "why isn't
this showing up," or anything similar, reconcile any dashboard against ground truth instead of
trusting it blindly:

1. List actual folders: `find "Resume Tailoring" -maxdepth 2 -type d`. Every folder containing
   a resume .docx is a processed role.
2. Read every row of `Job Applications Tracker.xlsx` (Company, Role, Status, Resume File, Job URL).
3. Read your dashboard's underlying data (if you have one).
4. Cross-check all sources and fix any mismatch — stale states, missing entries, stale job links.
5. Only after reconciling, answer the question or present the board. Never assume a dashboard
   is accurate just because it renders — it can drift from the tracker/folders over time.

### Applied Indicator (if using a dashboard)
Track whether you've actually applied, independent of resume/pipeline status:
- Fields: `applied: true/false`, `appliedDate: "YYYY-MM-DD"|null`, sourced from the tracker Status
  column. Status ∈ {Applied, Phone Screen, Interview, Offer, Rejected, Withdrawn, ...} → `applied: true`.
  Status ∈ {Not Applied, Identified} → `applied: false`.
- Whenever Step 6 or "Updating an Existing Application" changes a tracker Status to anything
  indicating a submitted application, also flip that role's `applied`/`appliedDate` in the same pass.

### Step 8 — Present Files to User
Share the PDF, the resume .docx, and the cover letter or introductory email .docx from Step 5a.
End with one sentence: what role, what company, what you emphasized.

---

## Formatting Rules (Applied to Every Tailored Resume)

Customize these to match your own resume style — these are just a sensible default:

- Font: pick one font and use it throughout
- Name: 18pt
- Major section headings: 14pt, ALL CAPS (e.g. PROFESSIONAL EXPERIENCE)
- Headline: 14pt, ALL CAPS (e.g. [YOUR TARGET ROLE FAMILY, e.g. PRODUCT MANAGEMENT LEADER])
- Company names: 13pt Bold
- Job titles: 12pt Bold
- Body text: 12pt
- Margins: 0.6" all sides
- Max length: two pages
- Left-justified throughout
- No underline bars or horizontal rules
- Bold titles and headings; dates are NOT bold
- Contact line (city, phone, email, LinkedIn): 10pt
- Company | City, State  (pipe with spaces on both sides)
- Job Title | Date Range  (pipe with spaces on both sides)
- Single space between all words and pipes

---

## Tracker Status Values

External roles: Not Applied → Applied → Phone Screen → Interview → Offer / Rejected / Withdrawn

Internal roles (if applicable — e.g. applying within your current company):
Identified → Info Interview Requested → Info Interview Scheduled → First Round Interview → Offer / Not Interested / Withdrawn

The tracker has a Source column (Internal / External). Always pass the correct source as the 8th argument to update-tracker (default is "External"):
```bash
python pipeline_helper.py update-tracker \
  "<Company>" "<Role>" "<YYYY-MM-DD>" \
  "<resume_file>" "<job_url>" "<status>" "<source>"
```

Update the tracker Status column when you report progress on an application.

---

## Internal Job Pipeline (Optional Pattern)

If your current employer posts internal roles via email notifications, you can mirror the
external pipeline for them:

1. **Job description**: Search for the role title + your company name on the public careers
   site to get the full JD. If not found, extract details from the email body.
2. **Folder & PDF**: Run Steps 1–3 same as the external pipeline.
3. **Resume**: Run the full pipeline automatically, same as external roles — generate the
   tailored resume (Steps 4–5) without waiting to be asked. Even though internal roles are
   usually applied to via an internal tool rather than a submitted resume, the resume here
   mainly supports the introductory-email step below and any informational interview.
3a. **Introductory Email**: Generate the short introductory email per Step 5a above
    (`Introductory Email - <Company> - <Role>.docx`) — states you're requesting an
    informational interview and references the attached resume.
4. **Tracker**: Add with `source = "Internal"` and initial status `"Identified"`.
5. **Status updates**: map your company's internal process stages to the status ladder above.
6. **Present**: Share the job description PDF, the resume, and the introductory email.

---

## Target Roles & Salary Threshold

Define your own rules here, for example:

Always process (no salary threshold):
- [Your top-priority role 1, e.g. Principal Product Manager]
- [Your top-priority role 2, e.g. Director of Engineering]
- [Equivalent senior level at major companies, e.g. Google L7, Meta E7]

Process only if salary top of range ≥ [Your Target Salary Threshold]:
- [Your conditional role type, e.g. Senior PM / Senior Engineer]
- Any "Senior" variant of your target role family

If salary is not mentioned for a conditional role, surface it to yourself and decide manually
rather than guessing — never assume it clears the bar.

Skip without asking:
- Roles clearly below your target level (e.g. individual-contributor titles, junior variants)
- Roles requiring deep domain expertise you don't have, where the JD lists it as a basic
  qualification (fill in your own dealbreaker domains here)

## Job Scoring (Optional — for evaluating whether a lead is worth pursuing)

Before running the full pipeline on a role, score it so effort goes only where it's likely to
pay off. This applies whether you're evaluating a lead from a scan or one you found yourself.

### Your Profile

Fill this in once so scoring has something to compare against:
- **Current role**: [Your current title, company, location]
- **Experience**: [Years and domain — e.g. "12+ years in product management — fintech, payments platforms"]
- **Strengths**: [Your top 3-5 differentiators]
- **Education**: [Degrees, if relevant to the roles you're targeting]
- **Location**: [Your base location and remote preference]

### Scoring Dimensions (1–5 each)

Score each dimension with a brief rationale — don't just assign numbers.

| Dimension | What it measures |
|-----------|-------------------|
| **Salary fit** | Does comp meet/exceed your target threshold? 5 = well above, 4 = above, 3 = borderline/unconfirmed, 2 = top of range barely clears, 1 = clearly below |
| **Level match** | Is the title/level equivalent to your target level? 5 = exact match, 4 = close (e.g. one tier adjacent), 3 = one step off, 2 = lateral, 1 = step down |
| **Scope match** | Program/project scale, cross-functional complexity, team or org breadth — how well does it match your career level? |
| **Technical fit** | How well do the specific technical domains align with your background? |
| **Domain fit** | Industry alignment — your strongest industries score highest; industries outside your experience score lowest |

**Overall score** = average of the 5 dimensions, rounded to 1 decimal.

### Verdicts

- **4.0–5.0 → Process**: Recommend running the full pipeline (JD PDF + tailored resume + tracker entry)
- **3.0–3.9 → Review**: Surface the key concerns and let yourself decide
- **Below 3.0 → Skip**: Not recommended — explain the gaps clearly and briefly

### Scoring Steps

1. **Get the JD** — if a URL is provided, fetch it. If it's a job board, find the company's own
   careers page for the canonical version. If just a snippet or role name, use what's available
   and note any assumptions.
2. **Extract key facts**: company, role title, location, salary range (or "not listed"), level
   indicator, key requirements.
3. **Apply the threshold check** — before scoring, verify the role meets your target criteria
   above. If it's a conditional-tier role with no salary listed, stop and ask yourself rather
   than guessing.
4. **Score each dimension** with a one-line rationale per dimension.
5. **Produce the scorecard** using this format:

```
**[Company] — [Role Title]**
Salary: [range or "Not listed — confirm before applying"]
Location: [location]

| Dimension     | Score |
|---------------|-------|
| Salary fit    | X/5   |
| Level match   | X/5   |
| Scope match   | X/5   |
| Technical fit | X/5   |
| Domain fit    | X/5   |
| **Overall**   | **X.X / 5** |

**Verdict: [Process / Review / Skip]**

✓ [Strongest alignment point]
✓ [Second strength]
− [Key gap or concern]
[− Second gap if significant]

[One sentence recommendation]
```

6. **Update your dashboard, if you use one** — add a new card for this role, following the same
   pattern as existing roles (e.g. distinct styling for Process vs. Review vs. Skip).
7. **Offer next step**:
   - Process verdict → "Want me to run the full pipeline for this role?"
   - Review verdict → "Want me to flag this for consideration, or run the pipeline anyway?"
   - Skip → no pipeline offer needed

### What Not to Do When Scoring

- Don't invent salary data — if it's missing, say so
- Don't auto-run the pipeline off a score; always ask first
- Don't produce a full scorecard for roles clearly below your target level — just say they fall
  below the threshold

## Job Discovery (Optional — if scanning email for leads)

If you want Claude to proactively scan your inbox for matching roles, define your own search
queries, for example:

1. `"[Your Target Title 1]" OR "[Your Target Title 2]" newer_than:30d`
2. `subject:(job OR opportunity OR role OR position OR hiring) ("[keyword]") newer_than:30d`
3. `from:[internal-job-notifications-address] newer_than:30d`  ← if your employer posts internal roles this way

For each result, extract: Company, Role Title, Date, Salary (if listed), Brief snippet.
Flag roles where no salary is mentioned so you can decide whether to process them.
Skip obvious spam, newsletters, or roles clearly below your target level.

Present results as a table: Company | Role | Date | Salary | Action, then ask which ones to
run through the full pipeline (or let it run automatically — your call).

---

## Updating an Existing Application
If you applied, got a screen, got an interview, or got a result:
1. Open `Job Applications Tracker.xlsx`
2. Find the matching row (company + role)
3. Update the Status cell and add a note with the date
4. Save and review the file
5. If using a dashboard artifact, update the matching card: set `applied: true` and
   `appliedDate` to the date if the new status indicates an application was submitted, and
   update `state`/`verdict` if relevant (e.g. moving toward "interview"). Do this in the same
   turn — don't wait for a sync check to catch it.

## Base Resume Location
Set this to wherever you keep your master resume, e.g.:
`Resume Tailoring/Base Resume - [Your Name].docx`
