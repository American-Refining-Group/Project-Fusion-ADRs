# User Acceptance Testing (UAT) Process

## Objective
Ensure delivered functionality meets the Business Requirements Document (BRD) and is free from critical defects before go-live.

---

## Roles & Responsibilities
- **Business Users** – Execute test cases, report issues, provide feedback.
- **Internal ARG-QA** – Bridge between Users and Consultant; verifies features before UAT, filters bugs, manages communication.
- **Consultant QA** – Logs validated bugs into JIRA, investigates, resolves, and updates status.

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
   - Classify as: bug, misunderstanding, or enhancement request  

5. **UAT Review Meeting**  
   - ARG-QA and Users meet to confirm classifications and finalize the bug list.

6. **Bug Submission to DAMCO QA**  
   - ARG-QA submits validated bugs to DAMCO QA, who logs them in **JIRA** and begins resolution.

7. **Bug Resolution & Tracking**  
   DAMCO QA fixes issues, marks them as **Resolved** in JIRA, and assigns back to ARG-QA for validation.

8. **Bug Validation & UAT Sign-Off**  
   ARG-QA and Users re-test resolved items. UAT completes when:  
   - All critical/high defects are fixed and verified  
   - No open medium/low defects block go-live  
   - User sign-off is received

---

## Special Handling of Challenges
- **Bug vs. Misunderstanding** – ARG-QA verifies against BRD; if behavior matches, it’s not a bug.
- **Different Expectations** – Document as enhancement requests; not part of current UAT scope.
- **Additional Functionality Requests** – Mark in spreadsheet for future review; exclude from defect count.
- **Backend Data Verification** – ARG-QA coordinates with DAMCO/internal IT for database checks (e.g., receipt numbers, gallons).

---

## Tools & Communication
- **OneDrive Spreadsheet** – Centralized logging of all user-reported issues.
- **JIRA** – DAMCO QA’s bug tracking system.
- **Email / Teams** – Communication between ARG-QA, Users, and DAMCO QA.
