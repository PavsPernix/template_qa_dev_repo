# QA + Dev Workflow Protocol

This document describes the complete QA + Dev collaboration workflow used in this repository. It covers the issue lifecycle, label conventions, automation, and how AI agents can interact with the system.

---

## Table of Contents

1. [Workflow Overview](#workflow-overview)
2. [Issue Types and Templates](#issue-types-and-templates)
3. [Label Conventions](#label-conventions)
4. [Project Board Fields](#project-board-fields)
5. [Workflow Stages](#workflow-stages)
6. [Automation (GitHub Actions)](#automation-github-actions)
7. [AI Agent Interaction Guide](#ai-agent-interaction-guide)
8. [Project Board Views](#project-board-views)

---

## Workflow Overview

```
Developer Flow:
  Backlog > To Do > In Progress > Ready for QA
  (Dev works on tasks, creates PR, merges)
  (Adds label "qa:ready" to User Story)

QA Flow:
  Ready for QA > In QA
  (Test cases auto-activate with Test Result = Pending)
  (QA tests each case, adds "qa:passed" or "qa:failed" label)

  If ALL test cases passed:
    > User Story auto-moves to "Ready for UAT"
  If any test case failed:
    > QA creates Bug sub-issue under User Story
    > Comment posted on User Story with failure details
    > User Story stays in "In QA" until dev fixes and QA re-tests

  Re-testing:
    > Dispatch "QA Reset" workflow to reset all test results to Pending

UAT Flow:
  Ready for UAT > In UAT > Ready for Deployment > Done
  (Product owner / stakeholders validate)
```

---

## Issue Types and Templates

### User Story (`[US]`)
- **Purpose**: Describes a feature from the user's perspective
- **Template**: `.github/ISSUE_TEMPLATE/user-story.yml`
- **Auto-label**: `user-story`
- **Structure**: User story format (As a... I want... So that...) with BDD acceptance criteria
- **Sub-issues**: Contains Task and Test Case sub-issues

### Test Case (`[TC]`)
- **Purpose**: Defines a specific test scenario for QA validation
- **Template**: `.github/ISSUE_TEMPLATE/test-case.yml`
- **Auto-label**: `test-case`
- **Structure**: Preconditions, test steps table, expected results
- **Parent**: Always linked as a sub-issue of a User Story

### Bug Report (`[BUG]`)
- **Purpose**: Reports a defect found during testing
- **Template**: `.github/ISSUE_TEMPLATE/bug-report.yml`
- **Auto-label**: `bug`
- **Structure**: Steps to reproduce, expected vs actual behavior, severity, environment

### Task (`[TASK]`)
- **Purpose**: A specific development task to implement a User Story
- **Template**: `.github/ISSUE_TEMPLATE/task.yml`
- **Auto-label**: `task`
- **Structure**: Task description, done criteria checklist
- **Parent**: Always linked as a sub-issue of a User Story

---

## Label Conventions

### Type Labels
| Label | Color | Purpose |
|-------|-------|----------|
| `user-story` | Blue (#1D76DB) | User stories |
| `test-case` | Green (#0E8A16) | Test cases |
| `bug` | Red (#D73A4A) | Bugs |
| `task` | Yellow (#FBCA04) | Dev tasks |

### Priority Labels
| Label | Color | Level |
|-------|-------|-------|
| `priority:critical` | Dark Red (#B60205) | P0 - System down, data loss |
| `priority:high` | Orange (#D93F0B) | P1 - Major feature broken |
| `priority:medium` | Yellow (#FBCA04) | P2 - Feature partially broken |
| `priority:low` | Light Blue (#C5DEF5) | P3 - Minor issue, cosmetic |

### Area Labels
| Label | Color | Scope |
|-------|-------|-------|
| `frontend` | Light Blue (#BFD4F2) | UI/UX components |
| `backend` | Light Purple (#D4C5F9) | Server-side logic |
| `api` | Light Green (#C2E0C6) | API endpoints |

### QA Workflow Labels
These labels drive the automation system:

| Label | Color | Trigger |
|-------|-------|----------|
| `qa:ready` | Blue (#0075CA) | Added to User Story when dev is done. Triggers test case sync. |
| `qa:in-progress` | Yellow (#E4E669) | QA is actively testing (manual status). |
| `qa:passed` | Green (#0E8A16) | Added to Test Case when test passes. Updates project field. |
| `qa:failed` | Red (#D73A4A) | Added to Test Case when test fails. Notifies parent story. |
| `qa:blocked` | Pink (#F9D0C4) | Added to Test Case when it cannot be tested. |

---

## Project Board Fields

### Status (Built-in)
| Status | Phase | Description |
|--------|-------|-------------|
| Backlog | Planning | Not yet prioritized |
| To Do | Planning | Prioritized for current/next sprint |
| In Progress | Development | Developer actively working |
| Ready for QA | Handoff | Dev done, waiting for QA |
| In QA | Testing | QA actively testing |
| Ready for UAT | Handoff | All tests passed, waiting for UAT |
| In UAT | Validation | Stakeholders validating |
| Ready for Deployment | Release | Approved, waiting for deploy |
| Done | Complete | Deployed and verified |
| Cancelled | Terminal | Will not be done |

### Type (Single Select)
- User Story
- Bug
- Test Case

### Test Result (Single Select) - Test Cases only
| Value | Color | Meaning |
|-------|-------|----------|
| Pending | Yellow | Not yet tested |
| Passed | Green | Test passed |
| Failed | Red | Test failed |
| Blocked | Orange | Cannot test due to dependency |

### Priority (Single Select)
| Value | Color |
|-------|-------|
| P0 - Critical | Red |
| P1 - High | Orange |
| P2 - Medium | Yellow |
| P3 - Low | Gray |

### Other Fields
- **Estimate**: Story points or time estimate
- **Sprint**: Iteration tracking

---

## Workflow Stages

### 1. Story Creation
1. Product owner creates a User Story using the template
2. Story is auto-added to the project board (Backlog)
3. Team reviews and estimates during sprint planning

### 2. Task Breakdown
1. Dev lead creates Task sub-issues under the User Story
2. QA lead creates Test Case sub-issues under the User Story
3. Tasks and test cases are linked as sub-issues

### 3. Development
1. Developer moves story to "In Progress"
2. Developer works on tasks, creates branches, submits PRs
3. Tasks are closed as they're completed
4. When all tasks are done, developer adds `qa:ready` label to User Story

### 4. QA Testing
1. `qa:ready` label triggers automation:
   - All test case sub-issues get "Test Result = Pending"
   - All test cases move to "In QA" status
   - A checklist comment is posted on the User Story
2. QA engineer tests each test case
3. For each test case, QA adds the appropriate label:
   - `qa:passed` - Test passed
   - `qa:failed` - Test failed
   - `qa:blocked` - Cannot test

### 5. Result Processing
- When `qa:passed`/`qa:failed`/`qa:blocked` is added to a test case:
  - The "Test Result" project field is updated automatically
  - The parent User Story is checked for completion
- If **ALL** test cases have `qa:passed`:
  - User Story moves to "Ready for UAT"
  - Summary comment posted
- If **ANY** test case has `qa:failed`:
  - QA creates a Bug sub-issue under the User Story
  - Comment posted on User Story with failure details

### 6. Re-testing (QA Reset)
If bugs are fixed and re-testing is needed:
1. Dispatch the "QA Reset" workflow with the User Story number
2. All test case labels (`qa:passed`, `qa:failed`, `qa:blocked`) are removed
3. All test cases reset to "Test Result = Pending" and "Status = In QA"
4. Reset comment posted on User Story

### 7. UAT and Deployment
1. Product owner validates in UAT
2. If approved, moves to "Ready for Deployment"
3. After deployment, moves to "Done"

---

## Automation (GitHub Actions)

### `auto-add-to-project.yml`
- **Trigger**: Any issue opened
- **Action**: Adds issue to project, sets Type field based on labels

### `qa-sync-test-cases.yml`
- **Trigger**: `qa:ready` label added to an issue
- **Action**: Finds test case sub-issues, sets them to Pending/In QA, posts checklist

### `qa-update-result.yml`
- **Trigger**: `qa:passed`, `qa:failed`, or `qa:blocked` label added
- **Action**: Updates Test Result field, checks sibling test cases, auto-transitions parent

### `qa-reset.yml`
- **Trigger**: Manual workflow dispatch with issue number
- **Action**: Resets all test case results and labels for re-testing

### Required Secret
All workflows require a `PROJECT_TOKEN` secret (GitHub PAT with `project`, `issues`, `repo`, and `workflow` scopes).

---

## AI Agent Interaction Guide

This system is designed to be agent-friendly. An AI agent can participate in the QA workflow through the following mechanisms:

### Reading (Query)
```bash
# Get all open issues
gh api repos/PavsPernix/template_qa_dev_repo/issues --jq '.[] | {number, title, labels: [.labels[].name]}'

# Get sub-issues of a user story
gh api repos/PavsPernix/template_qa_dev_repo/issues/8/sub_issues

# Check project field values via GraphQL
gh api graphql -f query='query { node(id: "ISSUE_NODE_ID") { ... on Issue { projectItems(first: 10) { nodes { fieldValues(first: 10) { nodes { ... on ProjectV2ItemFieldSingleSelectValue { name field { ... on ProjectV2SingleSelectField { name } } } } } } } } } }'
```

### Executing Test Cases
1. Read the test case issue body for steps and expected results
2. Execute the test (manual or automated)
3. Report results via labels and comments

### Reporting Results
```bash
# Mark test case as passed
gh issue edit 11 --repo PavsPernix/template_qa_dev_repo --add-label "qa:passed"

# Mark test case as failed
gh issue edit 11 --repo PavsPernix/template_qa_dev_repo --add-label "qa:failed"

# Post evidence as a comment
gh issue comment 11 --repo PavsPernix/template_qa_dev_repo --body "## Test Result: Passed

All steps executed successfully.

**Evidence:**
- Step 1: Login page loaded correctly
- Step 2: Credentials accepted
- Step 3: Redirected to dashboard
- Step 4: Username visible in header"
```

### Creating Bug Reports
```bash
gh issue create --repo PavsPernix/template_qa_dev_repo \
  --title "[BUG] Login fails with special characters in password" \
  --label "bug,priority:high,backend" \
  --body "## Related Test Case
#11

## Bug Description
Login endpoint returns 500 when password contains special characters like & or <

## Steps to Reproduce
1. Navigate to /login
2. Enter email: test@example.com
3. Enter password: Pass&word<123
4. Click Sign In

## Expected Behavior
User should be authenticated successfully

## Actual Behavior
Server returns HTTP 500 Internal Server Error"
```

### Workflow Summary for Agents
1. **Read** test case issues to understand what to test
2. **Execute** tests (via API calls, UI automation, etc.)
3. **Report** by adding `qa:passed` or `qa:failed` labels
4. **Comment** with detailed evidence and logs
5. **Create bugs** if tests fail
6. The automation system handles all status transitions automatically

---

## Project Board Views

### Recommended Views

| View | Type | Filter/Group | Purpose |
|------|------|--------------|---------|
| Board | Board | Group by Status | Overall workflow overview |
| My Items | Table | Filter by Assignee | Personal task list |
| Sprint Backlog | Table | Filter by Sprint | Sprint planning |
| In QA - Test Cases | Table | Filter: label=test-case, Status=In QA | QA dashboard |
| Bugs | Table | Filter: label=bug, State=open | Bug tracking |
| By Priority | Board | Group by Priority | Priority-based view |

### "In QA - Test Cases" View
This is the most important view for QA:
- **Filter**: `label:test-case` AND `status:In QA`
- **Columns**: Title, Test Result, Priority, Assignee, Parent Issue
- **Sort**: By Priority (Critical first)
- Shows all active test cases with their current test results
