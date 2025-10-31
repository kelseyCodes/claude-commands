
# Monthly Performance Summary Command

Generate a comprehensive monthly summary of your GitHub contributions for performance tracking.

## Usage

- `/monthly-summary --repo=owner/repo` - Generate summary for current month and upload to GitHub
- `/monthly-summary 2025-10 --repo=owner/repo` - Generate summary for October 2025 and upload to GitHub
- `/monthly-summary --repo=owner/repo --username=otheruser` - Use different GitHub username
- `/monthly-summary --repo=owner/repo --local-only` - Save locally only, skip GitHub upload
- `/monthly-summary 2025-09 --repo=owner/repo --username=otheruser --local-only` - Combine options

## Task

You are tasked with creating a monthly PR summary. Follow these steps:

### 1. Parse Arguments & Setup

Extract the month/year and repository from the command:
{{RAW_PROMPT}}

- If no month/year provided, use the current month
- Parse `--repo=OWNER/REPO` flag (REQUIRED)
- Parse `--username=X` flag if provided
- Parse `--local-only` flag if provided (default: false, meaning upload to GitHub by default)
- Otherwise get GitHub username from: `git config user.github`
- If git config is empty, ask the user for their GitHub username
- If no repo provided, ask the user for the repository in format: "What repository would you like to generate the summary for? (format: owner/repo)"
- Calculate date range: first day to last day of the specified month (format: YYYY-MM-DD)

### 2. Ensure Directory Exists

Create the directory if it doesn't exist:

```bash
mkdir -p .monthly-summary
```

### 3. Fetch PR Data

Use the GitHub CLI to fetch:

**PRs authored by the user:**

```bash
gh pr list --repo OWNER/REPO --author=USERNAME --search "created:START_DATE..END_DATE" --state=all --limit=100 --json number,title,state,url,createdAt,mergedAt,additions,deletions,files,labels,body
```

**PRs reviewed by the user (excluding their own):**

```bash
gh pr list --repo OWNER/REPO --search "reviewed-by:USERNAME -author:USERNAME created:START_DATE..END_DATE" --state=all --limit=100 --json number,title,url,state
```

### 4. Analyze PRs

For each authored PR:

- Parse the PR body for business impact mentions
- Extract linked tickets (JIRA format like `[TICKET-123]`, or common prefixes like `UC-123`, `HL-123`, `PROJ-123`, etc.)
- Identify project area from file paths in the `files` field:
  - Look for common directory patterns (e.g., `src/`, `app/`, `lib/`, `packages/`, `modules/`)
  - Group by top-level directories or feature areas
  - `db/migrate/` or `migrations/` → Database/Infrastructure
  - `spec/`, `test/`, `tests/` → Testing Infrastructure
  - `docs/` → Documentation
  - Other patterns → infer from directory names
- Categorize by type using PR title, labels, and body:
  - Features (new functionality, enhancements)
  - Bug Fixes
  - Refactoring
  - Infrastructure/DevOps
  - Testing
  - Documentation

### 5. Research Context (Optional but Preferred)

For major PRs or those with linked tickets:

- Search `.knowledge/` directory for related documentation (if it exists)
- Look for ADRs (Architecture Decision Records) if mentioned
- Check if there are related PRs mentioned in the description

### 6. Generate Report

Create a comprehensive markdown report with this structure:

```markdown
# Monthly Performance Summary - [Month Year]

## Executive Summary

[3-5 bullet points highlighting key achievements and business impact. Focus on outcomes, not just tasks completed. Examples:

- Implemented automated lead time management system, reducing manual intervention by 80%
- Fixed critical checkout bug affecting 5% of transactions
- Improved test coverage across vendor pack by 15%]

## Statistics

- **PRs Authored**: X (Y merged, Z open)
- **PRs Reviewed**: X
- **Total Lines Changed**: +X / -Y
- **Files Modified**: X
- **Primary Focus Areas**: [Top 2-3 project areas]

## Contributions by Area

### [Project Area 1] (X PRs)

#### Features ✨

- **[#123 - PR Title](PR_URL)** ✅ MERGED

  - Brief description of what was built
  - Business impact if mentioned
  - Key technical details
  - Linked tickets: [TICKET-123]

- **[#456 - PR Title](PR_URL)** 🔄 OPEN
  - ...

#### Bug Fixes 🐛

- ...

#### Refactoring ♻️

- ...

#### Infrastructure 🔧

- ...

### [Project Area 2] (X PRs)

[Same structure as above]

## Code Reviews 👀

- **[#789 - PR Title](PR_URL)** - Brief note about the review if significant
- **[#790 - PR Title](PR_URL)**
- ...

---

_Generated on [Date] using /monthly-summary_
```

### 7. Save Locally

- Save the report to: `.monthly-summary/YYYY-MM-monthly-summary.md`

### 8. Upload to GitHub (Default)

**If `--local-only` flag IS provided:**
- Skip GitHub upload entirely
- Print: `✅ Summary saved to: .monthly-summary/YYYY-MM-monthly-summary.md`

**If `--local-only` flag is NOT provided, attempt upload to private GitHub repository:**

1. **Check if repository exists:**
   ```bash
   gh repo view USERNAME/monthly-summaries >/dev/null 2>&1
   ```

2. **If repository doesn't exist, create it:**
   ```bash
   gh repo create USERNAME/monthly-summaries --private --description "Monthly performance summaries for tracking contributions"
   ```

3. **Clone or update repository:**
   ```bash
   # If directory doesn't exist, clone it
   if [ ! -d "/tmp/monthly-summaries" ]; then
     gh repo clone USERNAME/monthly-summaries /tmp/monthly-summaries
   else
     cd /tmp/monthly-summaries && git pull origin main
   fi
   ```

4. **Copy file to repository:**
   ```bash
   cp .monthly-summary/YYYY-MM-monthly-summary.md /tmp/monthly-summaries/YYYY-MM-monthly-summary.md
   ```

5. **Commit and push:**
   ```bash
   cd /tmp/monthly-summaries
   git add YYYY-MM-monthly-summary.md
   git commit -m "Add monthly summary for YYYY-MM"
   git push origin main
   ```

6. **If upload successful:**
   - Delete local file: `rm .monthly-summary/YYYY-MM-monthly-summary.md`
   - Print: `✅ Summary uploaded to: https://github.com/USERNAME/monthly-summaries/blob/main/YYYY-MM-monthly-summary.md`

7. **If upload fails at any step:**
   - Keep local file
   - Print error message explaining what failed
   - Print: `⚠️ Upload failed. Summary saved locally to: .monthly-summary/YYYY-MM-monthly-summary.md`

## Important Notes

- Use emojis for status: ✅ MERGED, 🔄 OPEN, ❌ CLOSED (not merged)
- Group PRs by project area first, then by category within each area
- In the executive summary, focus on **business value and impact**, not just technical tasks
- If a PR has significant complexity or impact, provide more detail
- Keep descriptions concise but informative
- Include PR numbers and links for easy reference
- If no PRs found for the month, still create a file with a message indicating no activity

## Error Handling

- If `gh` CLI is not installed, provide clear error message
- If authentication fails, provide instructions to run `gh auth login`
- If no PRs found, create report indicating no activity that month
- If git config user.github is empty and no --username provided, ask user for username
- If no --repo parameter provided, ask user for the repository
- **GitHub upload failures**: If any step of the GitHub upload process fails, keep the local file and print the local path with an error message
- If repository creation fails, explain the error and save locally
- If git push fails (e.g., permissions issue), explain the error and save locally
- If clone fails, provide instructions to check GitHub authentication and save locally
- **Only delete local file if the entire GitHub upload process succeeds**
