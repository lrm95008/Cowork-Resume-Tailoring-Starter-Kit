---
name: job-score
description: Score any job opportunity against [Your Name]'s profile and decide whether to pursue it. Use this skill whenever [Your Name] shares a job URL, pastes a job description, mentions a role she's seen, or asks "should I go for this?" or "what do you think about this role?" Evaluates salary fit, level match, scope match, technical fit, and domain fit on a 1-5 scale, applies the salary threshold rule for conditional roles, and produces a clear Process / Review / Skip recommendation. After scoring, updates the job-match-analysis artifact with the new role card.
---

# Job Match Scoring

## [Your Name]'s Profile

- **Current role**: [Your current title, company, location]
- **Experience**: [Years and domain -- e.g. "15+ years in technical program management -- consumer devices, streaming media"]
- **Strengths**: [Your top 3-5 differentiators]
- **Education**: [Degrees, if relevant to the roles you're targeting]
- **Location**: [Your base location and remote preference]

## Target Role Criteria

**Always process (no salary threshold):**
- [Your top-priority role 1 -- e.g. Principal TPM / Principal Technical Program Manager]
- [Your top-priority role 2 -- e.g. Director of Program Management]
- [Equivalent senior levels at major companies -- e.g. Google L7, Meta E7]

**Process only if salary top of range >= [Your Target Salary Threshold]:**
- [Your conditional role type -- e.g. Senior TPM / Senior Technical Program Manager]
- Any "Senior" variant of your target role family

**If salary not listed for a conditional role:** Surface it to [Your Name] -- tell them the salary is missing and ask whether to proceed.

**Skip without asking:**
- Roles clearly below your target level (e.g. Program Manager, Project Manager, Associate variants)
- Roles requiring deep domain expertise you don't have, where the JD lists it as a basic qualification (fill in your own dealbreaker domains)

## Scoring Dimensions (1-5 each)

Score each dimension with a brief rationale -- don't just assign numbers.

| Dimension | What it measures |
|-----------|-----------------|
| **Salary fit** | Does comp meet/exceed your threshold? 5 = well above, 4 = above, 3 = borderline/unconfirmed, 2 = barely clears, 1 = clearly below |
| **Level match** | Is the title/level equivalent to your target? 5 = exact match, 4 = close (one tier adjacent), 3 = one step off, 2 = lateral, 1 = step down |
| **Scope match** | Program/project scale, cross-functional complexity, team or org breadth -- how well does it match your career level? |
| **Technical fit** | How well do the specific technical domains align with your background? |
| **Domain fit** | Industry alignment -- your strongest industries score highest; industries outside your experience score lowest |

**Overall score** = average of the 5 dimensions, rounded to 1 decimal.

## Verdicts

- **4.0-5.0 -> Process**: Recommend running the full pipeline
- **3.0-3.9 -> Review**: Surface the key concerns and let [Your Name] decide
- **Below 3.0 -> Skip**: Not recommended -- explain the gaps clearly and briefly

## Steps

1. **Get the JD** -- if a URL is provided, fetch it. If it's a job board, find the company's own careers page for the canonical version. If just a snippet or role name, use what's available and note any assumptions.

2. **Extract key facts**: company, role title, location, salary range (or "not listed"), level indicator, key requirements.

3. **Apply threshold check** -- before scoring, verify the role meets the target criteria above. If it's a conditional-tier role with no salary listed, stop and ask [Your Name].

4. **Score each dimension** with a one-line rationale per dimension.

5. **Produce the scorecard** using this exact format:

    **[Company] -- [Role Title]**
    Salary: [range or "Not listed -- confirm before applying"]
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

    [Strongest alignment point]
    [Second strength]
    [Key gap or concern]
    [Second gap if significant]

    [One sentence recommendation]

6. **Update the job-match-analysis artifact** -- read the current artifact HTML (artifact id: job-match-analysis), add a new card for this role following the same card pattern as existing roles (featured green border for Process, standard border for Review, dim styling for Skip), and call mcp__cowork__update_artifact.

7. **Offer next step**:
   - Process verdict -> "Want me to run the full pipeline for this role?"
   - Review verdict -> "Want me to flag this for consideration, or run the pipeline anyway?"
   - Skip -> No pipeline offer needed

## What not to do

- Don't invent salary data -- if it's missing, say so
- Don't auto-run the pipeline; always ask first
- Don't score roles clearly below the target level -- tell [Your Name] these fall below the threshold and don't need a full scorecard
