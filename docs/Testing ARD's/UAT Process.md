# User Acceptance Testing (UAT) Process

## Objective
Ensure delivered functionality meets the Business Requirements Document (BRD) and is free from critical defects before go-live.

---

## Roles & Responsibilities
- **Business Users** – Execute test cases, report issues, provide feedback.
- **Internal ARG-QA** – Bridge between Users and Consultant; verifies features before UAT, filters bugs, manages communication, and performs triage classification.
- **Consultant QA (DAMCO)** – Logs validated bugs into JIRA, investigates, resolves, and updates status.

---

## Process Flow
1. **Smoke Test (DAMCO QA)**  
   DAMCO QA verifies the environment is stable and major functionality works. Critical blockers are fixed before proceeding.

2. **Pre-UAT Verification (ARG-QA)**  
   ARG-QA performs a quick functional check to confirm:  
   - Feature is accessible in UAT environment  
   - Main buttons/links work  
   - No blocking errors occur  
   - Core workflows are usable for testing  

3. **User Testing & Logging Issues**  
   Users execute test scenarios and log all issues (bugs, usability concerns, questions) into a shared **OneDrive spreadsheet** with:  
   - Steps taken  
   - Expected vs. actual results  
   - Screenshots/data references

4. **ARG-QA Review & Triage**  
   ARG-QA reviews entries to:  
   - Remove duplicates  
   - Identify misunderstandings  
   - Classify using the **Triage Classification System** (see below)

5. **UAT Review Meeting**  
   - ARG-QA and Users meet to confirm classifications and finalize the issue list based on triage categories.

6. **Issue Resolution Based on Triage**  
   Issues are handled according to their triage classification:
   - **Clear and Communicate Immediately**: 
      - Category 1 (User Mis-Interpretation)
   - **Send Directly to DAMCO**: 
      - Category 2 (Design Adherence)
      - Category 3 (Clear Defect)
   - **Requires ARG Internal Review**: 
      - Category 4 (Business Rule Implementation Error)
      - Category 5 (Enhancement Request)

7. **Bug Resolution & Tracking**  
   DAMCO QA fixes issues, updates JIRA status, and coordinates with ARG-QA for validation.

8. **Bug Validation & UAT Sign-Off**  
   ARG-QA and Users re-test resolved items. UAT completes when:  
   - All critical/high defects are fixed and verified  
   - No open medium/low defects block go-live  
   - User sign-off is received

---

## Triage Classification System

| Classification | Description | Action Required |
|---|---|---|
| **1 - User Mis-Interpretation** | User misunderstood functionality or process | Clear Immediately with user education |
| **2 - Design Adherence** | System not following approved design specifications | Send Directly to DAMCO |
| **3 - Clear Defect** | Obvious system malfunction or error | Send Directly to DAMCO |
| **4 - Business Rule Implementation Error** | System not implementing business rules correctly | Requires ARG Internal Review |
| **5 - Enhancement Request** | New functionality or improvement request | Requires ARG Internal Review |

---

## JIRA Status Tracking

| JIRA Status | Description | Required Action |
|---|---|---|
| **In-Progress** | DAMCO actively working on the issue | Monitor progress |
| **ARG-QA Input Required** | DAMCO requires clarification from ARG | ARG provides requested information |
| **Implemented** | Fix deployed to UAT environment | **REQUIRES ACTION:** - ARG-QA Re-test and validate |
| **Resolved No Implementation** | Issue resolved without code changes | ARG-QA Needs to confirm and mark done |
| **Done** | ARG has validated and closed the issue | Update tracking spreadsheet |



## Tools & Communication
- **OneDrive Spreadsheet** – Centralized logging of all user-reported issues with triage classifications.
- **JIRA** – DAMCO QA's bug tracking system with standardized status workflow.
- **QA Meetings / Teams** – Communication between ARG-QA, Users, and DAMCO QA.

---

## Workflow Summary by Triage Category

**Category 1 (User Mis-Interpretation):**
- OneDrive Entry → ARG-QA Review → Clear Immediately → User Communication

**Categories 2 & 3 (Design Adherence / Clear Defect):**
- OneDrive Entry → ARG-QA Review → Direct DAMCO Submission → JIRA Tracking → Resolution → User Validation

**Categories 4 & 5 (Business Rule Error / Enhancement):**
 - OneDrive Entry → ARG-QA Review → ARG Internal Review → Decision → Potential DAMCO Submission → JIRA Tracking (if applicable)