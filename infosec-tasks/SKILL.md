---
name: infosec-tasks
description: Retrieve and filter Infosec tasks from ClickUp by Quarter. Use when asked to pull security engineering tasks, GRC tasks, penetration testing tasks, or roadmap items filtered by fiscal quarter (e.g., "FY25 Q4 tasks", "what's on the Q1 roadmap").
---

# Infosec Tasks Skill

Retrieves tasks from the Infosec ClickUp space and filters by Quarter custom field.

## Configuration

**Space:** Infosec
**Space ID:** `30075124`
**Workspace ID:** `2323726`

**Target Lists:**
| List | ID |
|------|-----|
| Security Engineering | `132251242` |
| Penetration Testing | `900202119210` |
| GRC | `901704270170` |

**Quarter Custom Field:**
- Field ID: `df8ff1a4-7dd1-4129-b3f6-cf0b745a21c9`
- Type: dropdown

**Quarter Options:**
| Quarter | Value (orderindex) |
|---------|-------------------|
| FY26 Q1 | 0 |
| FY26 Q2 | 1 |
| FY26 Q3 | 2 |
| FY26 Q4 | 3 |
| FY25 Q4 | 4 |
| FY25 Q3 | 5 |
| FY27 Q1 | 6 |

## Usage

When the user asks for tasks filtered by quarter, follow this workflow:

### Step 1: Parse the requested quarter

Map user input to the Quarter field value:
- "FY25 Q3" or "Q3 FY25" → value 5
- "FY25 Q4" or "Q4 FY25" → value 4
- "FY26 Q1" or "Q1 FY26" or "Q1" (current FY) → value 0
- "FY26 Q2" or "Q2 FY26" or "Q2" → value 1
- "FY26 Q3" or "Q3 FY26" or "Q3" → value 2
- "FY26 Q4" or "Q4 FY26" or "Q4" → value 3
- "FY27 Q1" → value 6

### Step 2: Search for all tasks in Infosec space

Use `clickup_search` to get all tasks. **Important:** Do NOT use status filters here—fetch all tasks and filter by Quarter field client-side to avoid missing tasks with unexpected status values.

```
Tool: mcp__clickup__clickup_search
Parameters:
  filters:
    location:
      projects: ["30075124"]
    asset_types: ["task"]
  count: 200
```

**Pagination:** If `next_cursor` is returned, make additional requests with `cursor: <next_cursor>` until all results are retrieved. Do not assume the first page contains all tasks.

### Step 3: Fetch detailed task info and filter

For each task returned, fetch full details using `clickup_get_task` with `detail_level: "detailed"`.

Check the `custom_fields` array for the Quarter field:
```javascript
// Find Quarter field and check value
const quarterField = task.custom_fields.find(f => f.id === "df8ff1a4-7dd1-4129-b3f6-cf0b745a21c9");
if (quarterField && quarterField.value === targetQuarterValue) {
  // Task matches the requested quarter
}
```

### Step 4: Filter by target lists

Only include tasks from the three target lists:
- Security Engineering (`132251242`)
- Penetration Testing (`900202119210`)
- GRC (`901704270170`)

Check `task.list.id` to verify.

### Step 5: Format and present results

Group results by list and present in a table:

```markdown
## FY25 Q4 Infosec Tasks

### Security Engineering
| Task | Status | Assignees | Due Date |
|------|--------|-----------|----------|
| Task name | status | names | date |

### GRC
| Task | Status | Assignees | Due Date |
|------|--------|-----------|----------|
| Task name | status | names | date |

### Penetration Testing
| Task | Status | Assignees | Due Date |
|------|--------|-----------|----------|
| Task name | status | names | date |
```

## Optimization: Batch Processing

To reduce API calls, process tasks in batches of 5 parallel `clickup_get_task` calls.

## Example Invocation

User: "Show me the FY25 Q4 roadmap tasks"

Response workflow:
1. Identify quarter: FY25 Q4 → value 4
2. Search Infosec space for all tasks
3. Batch fetch task details (5 at a time)
4. Filter where Quarter field value === 4
5. Filter to only Security Engineering, GRC, Penetration Testing lists
6. Present grouped table

## Roadmap Snapshots

Snapshots capture the planned tasks for a quarter at a point in time. This enables tracking what was originally planned vs. what actually happened.

**Snapshot Directory:** `data/roadmap-snapshots/` (in project root)

### Snapshot JSON Schema

```json
{
  "quarter": "FY25 Q4",
  "quarter_value": 4,
  "snapshot_date": "2025-10-01T00:00:00Z",
  "tasks": [
    {
      "id": "86dxqxa9r",
      "name": "2025 External Pen Test",
      "list": "Security Engineering",
      "list_id": "132251242",
      "status": "Open",
      "assignees": ["John Clawson"],
      "due_date": "2025-01-16"
    }
  ],
  "total_count": 5
}
```

### Creating a Snapshot

When the user asks to "snapshot the roadmap" for a quarter:

#### Step 1: Parse the quarter
Map to quarter value (same as Usage section above).

#### Step 2: Pull all tasks for that quarter

**Critical:** Use the robust search workflow to ensure all tasks are captured:

1. **Search without status filters** - Do NOT filter by `task_statuses`. Fetch ALL tasks in the space, then filter by Quarter field value client-side. This prevents missing tasks with non-standard status values (e.g., "completed" vs "done").

2. **Paginate fully** - Check for `next_cursor` in search results and continue fetching until all pages are retrieved.

3. **Use high count** - Set `count: 200` to reduce pagination needs.

4. **Include all target lists** - Filter results to Security Engineering, GRC, and Penetration Testing lists by checking `task.list.id`.

```
Tool: mcp__clickup__clickup_search
Parameters:
  filters:
    location:
      projects: ["30075124"]
    asset_types: ["task"]
  count: 200
```

Then for each task, fetch details and check if `custom_fields` Quarter value matches the target quarter.

#### Step 3: Build snapshot object
For each task, extract:
- `id`: Task ID
- `name`: Task name
- `list`: List name (Security Engineering, Penetration Testing, GRC)
- `list_id`: List ID
- `status`: Current status
- `assignees`: Array of assignee names
- `due_date`: Due date if set

#### Step 4: Verify completeness

Before saving, confirm the task count with the user:
```
Found 23 tasks tagged with FY26 Q1:
- Security Engineering: 16 tasks
- GRC: 7 tasks
- Penetration Testing: 0 tasks

Does this match what you see in ClickUp?
```

If there's a discrepancy, ask the user to provide a screenshot of their ClickUp view to identify missing tasks.

#### Step 5: Save to file
Save to `data/roadmap-snapshots/{QUARTER}-snapshot.json`

File naming:
- "FY25 Q4" → `FY25-Q4-snapshot.json`
- "FY26 Q1" → `FY26-Q1-snapshot.json`

#### Example Command
User: "Snapshot the FY26 Q1 roadmap"

Result: Creates `data/roadmap-snapshots/FY26-Q1-snapshot.json`

### Comparing Snapshots to Current State

When comparing a snapshot to current task status (used by security-qbr skill):

#### Step 1: Load the snapshot
Read from `data/roadmap-snapshots/{QUARTER}-snapshot.json`

#### Step 2: Fetch current status of snapshot tasks
For each task ID in the snapshot, fetch current details using `clickup_get_task`.

#### Step 3: Fetch all tasks currently tagged for the quarter
Pull all tasks with the quarter tag (may include tasks added after snapshot).

#### Step 4: Categorize tasks

| Category | Definition |
|----------|------------|
| **Completed** | In snapshot AND now status = done/closed/completed |
| **Pushed** | In snapshot AND still open → slipped to next quarter |
| **Added & Completed** | NOT in snapshot, tagged for quarter, now done/closed/completed |
| **Added & In Progress** | NOT in snapshot, tagged for quarter, still open |

**Note:** Check for status values `"done"`, `"closed"`, AND `"completed"` when determining if a task is finished.

#### Step 5: Calculate completion rate
```
Completion Rate = Completed / Total in Snapshot × 100
```

Only tasks from the original snapshot count toward completion rate. Tasks added during the quarter are shown separately.

### Quarter Timing Reference

**Fiscal Year = Calendar Year**

| Quarter | Date Range | Previous Quarter |
|---------|------------|------------------|
| FY26 Q1 | Jan 1 - Mar 31, 2026 | FY25 Q4 |
| FY26 Q2 | Apr 1 - Jun 30, 2026 | FY26 Q1 |
| FY26 Q3 | Jul 1 - Sep 30, 2026 | FY26 Q2 |
| FY26 Q4 | Oct 1 - Dec 31, 2026 | FY26 Q3 |

**When to take snapshots:**
- Ideally at the start of each quarter
- Can be taken mid-quarter if needed (captures state at that point)

## Integration with security-qbr skill

This skill supports the `security-qbr` skill's agent-based architecture:

### Agent Support

| QBR Agent | How This Skill Helps |
|-----------|---------------------|
| Roadmap Agent | Snapshots, task status comparison |
| Highlights Agent | Closed tasks by theme |

---

### For Highlights Agent (QBR Section 3)

Extract engineering accomplishments from closed tasks, categorized by theme.

**Step 1: Search for closed tasks in the quarter**

Use `clickup_search` with status filter. **Important:** Include all closed statuses:
```
filters:
  task_statuses: ["done", "closed", "completed"]
  location:
    projects: ["30075124"]
  asset_types: ["task"]
count: 200
```

**Step 2: Filter to target lists**

Only include tasks from:
- Security Engineering (`132251242`)
- Penetration Testing (`900202119210`)
- GRC (`901704270170`)

Check `task.list.id` to verify.

**Step 3: Fetch task details**

For each task, use `clickup_get_task` with `detail_level: "detailed"` to get full context for categorization.

**Step 4: Categorize by theme**

Assign each task to one of these categories based on task name and description:

| Category | Keywords/Patterns |
|----------|-------------------|
| Identity & Access Management | user access review, SSO, IAM, identity, authentication, authorization, MFA |
| Monitoring & Detection | SIEM, logging, alerting, InsightIDR, detection, monitoring, phishing test |
| GRC & Compliance | audit, assessment, compliance, policy, SOC 2, ISO, risk assessment, GRC |
| Penetration Testing | pen test, penetration, vulnerability assessment, security testing |
| Security Operations | automation, tooling, process, runbook, incident response |
| Infrastructure Security | cloud security, network, firewall, Wiz, hardening, encryption |

**Step 5: Select top accomplishments**

For each category with tasks:
- Select 2-3 most impactful items
- Prefer tasks that:
  - Have clear business impact
  - Represent completed initiatives (not recurring tasks)
  - Are relevant to executive audience

**Output format for Highlights Agent:**

```json
{
  "quarter": "FY25 Q4",
  "generated_at": "2026-01-28T10:00:00Z",
  "categories": {
    "Identity & Access Management": [
      {"name": "Completed Quarterly User Access Reviews", "impact": "6 engineering teams audited"}
    ],
    "Monitoring & Detection": [
      {"name": "Integrated Workday logs into InsightIDR", "impact": "HR event visibility"}
    ],
    "GRC & Compliance": [],
    "Penetration Testing": [],
    "Security Operations": [],
    "Infrastructure Security": []
  },
  "total_highlights": 12
}
```

---

### For Roadmap Agent (QBR Section 2)

**Part 1: Past Quarter Accountability**

Compare snapshot to current state to show what was planned vs. completed.

1. Load snapshot from `data/roadmap-snapshots/{PREV_QUARTER}-snapshot.json`
2. For each task ID in snapshot, fetch current status via `clickup_get_task`
3. Categorize:
   - **Completed:** status = done/closed/completed
   - **Pushed:** status still open
4. Calculate completion rate: `Completed / Total × 100`
5. Fetch tasks currently tagged for quarter but NOT in snapshot (Added tasks)

**Part 2: Current Quarter Preview**

Get tasks planned for upcoming quarter.

1. Load snapshot from `data/roadmap-snapshots/{CURR_QUARTER}-snapshot.json` if exists
2. Or search ClickUp for tasks with Quarter = current quarter value
3. Group by list (Security Engineering, GRC, Penetration Testing)
4. Include: task name, status, assignees, due dates

---

### For QBR Accomplishments (legacy - use Highlights Agent instead)
Add status filter to Step 2. **Important:** Include `"completed"` in addition to `"done"` and `"closed"` as ClickUp treats these as separate status categories:
```
filters:
  task_statuses: ["done", "closed", "completed"]
  location:
    projects: ["30075124"]
  asset_types: ["task"]
```

### For QBR Upcoming Work (legacy - use Roadmap Agent instead)
Add status filter:
```
filters:
  task_statuses: ["active", "unstarted"]
  location:
    projects: ["30075124"]
  asset_types: ["task"]
```

### For QBR Roadmap Accountability
1. Load snapshot from `data/roadmap-snapshots/{QUARTER}-snapshot.json`
2. Compare to current task status
3. Generate accountability slide showing planned vs. actual

## Common Pitfalls & How to Avoid Them

### 1. Missing tasks due to status filters
**Problem:** Searching with `task_statuses: ["done", "closed"]` misses tasks with status `"completed"`.

**Solution:** Either include all closed statuses (`["done", "closed", "completed"]`) or search without status filters and filter client-side.

### 2. Incomplete results due to pagination
**Problem:** Search returns only first page of results, missing tasks beyond the `count` limit.

**Solution:** Always check for `next_cursor` in results and paginate until no cursor is returned.

### 3. Recently created tasks not appearing
**Problem:** Tasks created after the search was run won't be in the results.

**Solution:** When creating snapshots, verify the count matches expectations. If the user reports a discrepancy, re-run the search to catch recent additions.

### 4. Status category mapping
ClickUp has different status categories that may not align with intuitive filters:

| Status Category | Includes |
|-----------------|----------|
| `unstarted` | Open, next, pending action |
| `active` | in progress |
| `done` | done |
| `closed` | closed, completed |

**Best practice for snapshots:** Search without status filters, fetch all tasks, then filter by Quarter field value to ensure nothing is missed.
