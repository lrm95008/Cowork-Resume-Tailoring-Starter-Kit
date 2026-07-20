---
name: status-update
description: Update the status of a job application in the tracker. Use this skill whenever [Your Name] reports any progress on an application -- a call scheduled, interview confirmed, offer received, rejection, withdrawal, or any other milestone. Works for both external roles and internal roles (which have a different status ladder). Updates the Job Applications Tracker.xlsx, adds a dated note, and refreshes the Job Hunt Status Board artifact automatically. Trigger phrases include: "I have an interview", "I got rejected", "I withdrew from", "they moved me to", "I applied to", "phone screen with", "got an offer from", "not moving forward with", and similar updates about application progress.
---

# Application Status Update

## Status Ladders

Choose the correct ladder based on whether the role is Internal or External.

**External**: Not Applied -> Applied -> Phone Screen -> Interview -> Offer / Rejected / Withdrawn

**Internal** (roles at your current employer): Identified -> Info Interview Requested -> Info Interview Scheduled -> First Round Interview -> Offer / Not Interested / Withdrawn

### Mapping natural language to status values

| What [Your Name] says | External status | Internal status |
|---|---|---|
| "I applied" / "submitted my application" | Applied | -- |
| "I have a call" / "phone screen scheduled" | Phone Screen | Info Interview Scheduled |
| "Info interview requested" / "I reached out" | -- | Info Interview Requested |
| "I have an interview" / "interview scheduled" | Interview | First Round Interview |
| "I got an offer" | Offer | Offer |
| "I got rejected" / "not moving forward" / "passed on me" | Rejected | Not Interested |
| "I withdrew" / "no longer interested" | Withdrawn | Withdrawn |

## Tracker File

**Path**: [Your tracker file path -- e.g. Resume Tailoring/Job Applications Tracker.xlsx]

Active applications are on the **Applications** tab. Closed/inactive roles are on the **Inactive** tab.

**Column order** (0-indexed):
- 0: Company
- 1: Source (Internal / External)
- 2: Role
- 3: Date Added
- 4: Status  <-- update this
- 5: Job URL
- 6: Resume File
- 7: Notes  <-- append dated note here
- 8: Position ID

## Steps

1. **Identify the role** -- match the company and role fragment from the message to a row in the tracker. Use fuzzy matching (e.g., "Netflix" matches the Netflix TPM row). If genuinely ambiguous between two roles at the same company, ask to confirm.

2. **Determine the new status** -- use the mapping table above. Keep status values exactly as listed (capitalization matters for artifact badges).

3. **Update the tracker** with this Python snippet:

```python
import openpyxl
from datetime import date

wb = openpyxl.load_workbook('[TRACKER_PATH]')
ws = wb['Applications']

for row in ws.iter_rows(min_row=2):
    if row[0].value == 'COMPANY' and 'ROLE_FRAGMENT' in str(row[2].value or ''):
        row[4].value = 'NEW_STATUS'
        existing = row[7].value or ''
        new_note = f"{date.today().isoformat()}: NOTE_TEXT"
        row[7].value = (existing.rstrip(' |') + ' | ' + new_note).lstrip(' |')
        break

wb.save('[TRACKER_PATH]')
print("Updated")
```

Run via mcp__workspace__bash. Install openpyxl first if needed: pip install openpyxl --break-system-packages --quiet

4. **Refresh the Job Hunt Status Board artifact** -- read the updated tracker, rebuild the DATA JSON array, and call mcp__cowork__update_artifact with id job-hunt-status-board. Replace the const DATA = [...] line in the artifact HTML with fresh data.

5. **Confirm** -- briefly report what was updated:
   "[Company] [Role (shortened)] -> [New Status] (noted [date])"

6. **Suggest a natural next action** where it adds value:
   - Moving to Interview -> "Want me to pull up the JD and your tailored resume to prep?"
   - Offer received -> "Want to update the status of other active applications now that you have an offer?"
   - Rejected or Withdrawn -> "Want me to move this to the Inactive tab?"
   - Applied -> Nothing needed

## Edge cases

- **Role not in tracker**: Offer to run the full pipeline first to add it, then update the status
- **Internal roles**: Always use the Internal ladder -- even if the language used sounds like an external process
- **Multiple roles at same company**: Ask to confirm if there's any ambiguity
- **Backward status** (e.g., "actually I haven't applied yet"): Update without comment
- **"Closed" or "req closed"**: Move the row to the Inactive tab instead of updating status
