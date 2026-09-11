# QA Assessment — Task Management Application

**Name:** Aman Kumar Yadav  
**Email:** amanyadav9598037@gmail.com  
**Contact:** 8417079561  

---

## Context

This is a Task Management application where users can register, log in, and then create, view, edit, and delete their own tasks. Tasks are stored in a database and displayed as a list.

The application is close to production release and has no existing test documentation. So I had to think from scratch about what needs to be tested.

I have covered three areas:

- Test scenarios and test cases (positive, negative, edge)
- Input validation and error handling
- Bug and risk areas that can be identified without executing the application

---

## 1. Test Scenarios

### Registration

- A new user should be able to register with valid details.
- Registration should not be allowed with an already registered email.
- Blank or incorrectly formatted fields should show errors.
- Password and confirm password must match.
- Weak passwords should be rejected.
- Email should be case-insensitive (User@x.com and user@x.com treated the same).
- SQL injection or XSS in name/email should be handled safely.

### Login

- Login should succeed with valid credentials.
- Wrong password or unregistered email should show an error.
- Blank fields should show validation errors.
- Repeated wrong password attempts should trigger lockout or rate limiting.
- After session timeout, the user should be asked to log in again.
- Task pages should not be accessible without login.
- SQL injection in login fields should not work.

### Task CRUD

- A valid task should be created and appear in the list.
- Empty or over-length data should show an error.
- A user should only see their own tasks.
- A task should be editable, and cancelling should not save changes.
- Deletion should ask for confirmation, and cancelling should keep the task.
- Another user's task should not be accessible by changing the URL.
- Data should persist after a page refresh.
- If the database or network fails, the user should see a clean error message.

### Input Validation & Error Handling

- Required fields should be validated.
- Email, password, date, priority, and status should be validated.
- Max length and special characters should be handled.
- SQLi and XSS should be blocked.
- Server errors, session expiry, and offline scenarios should show friendly messages.
- Error messages should be field-specific and clear, without technical stack traces.

---

## 2. Test Cases

### Registration

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| REG-01 | Valid registration | Unique email + valid password + matching confirm password | Account is created; redirect to login/dashboard | Positive |
| REG-02 | Duplicate email | Register with an already registered email | "Email already registered" error; no duplicate account | Negative |
| REG-03 | Blank fields | Submit the form with all fields empty | Required field errors; no account created | Negative |
| REG-04 | Invalid email format | Enter `abc`, `a@b`, `a@b.c` | Format error; registration blocked | Negative |
| REG-05 | Password mismatch | Password `Test@123`, Confirm `Test@124` | Mismatch error | Negative |
| REG-06 | Weak password | Enter `12345` | Password policy error | Negative |
| REG-07 | Password length boundary | Enter minimum and maximum allowed length | Accepted within policy; rejected outside | Edge |
| REG-08 | Email case and spaces | Enter ` User@Example.com ` | Trimmed/normalized; duplicate check works correctly | Edge |
| REG-09 | Special characters in name | Enter `Rahul@#$%` or emoji | Allowed or a clear validation error is shown | Edge |
| REG-10 | SQL injection | Enter `' OR '1'='1` | Input sanitized; no DB error; no bypass | Security |
| REG-11 | XSS in name | Enter `<script>alert(1)</script>` | Script escaped; not executed | Security |

### Login

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| LOG-01 | Valid login | Registered email + correct password | Login succeeds; dashboard shown | Positive |
| LOG-02 | Wrong password | Correct email + wrong password | Invalid credentials error | Negative |
| LOG-03 | Unregistered email | Unknown email + any password | Invalid credentials error | Negative |
| LOG-04 | Blank fields | Submit with empty email/password | Required field errors | Negative |
| LOG-05 | Email case-insensitive | `User@x.com` vs `user@x.com` | Both work | Edge |
| LOG-06 | Password case-sensitive | Change the case of the correct password | Login fails | Edge |
| LOG-07 | SQL injection | Enter `' OR 1=1 --` | Login blocked; no bypass | Security |
| LOG-08 | Multiple failed attempts | Enter wrong password 5–10 times | Lockout or rate limiting applied | Negative |
| LOG-09 | Session timeout | Log in, wait for timeout, then perform an action | Redirect to login; session expired message | Edge |
| LOG-10 | Access without login | Open task URL directly without logging in | Redirect to login or 401/403 | Negative |

### Task CRUD — Create

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| CRT-01 | Valid task | Title, description, due date, priority, status | Task saved and visible in the list | Positive |
| CRT-02 | Empty title | Leave title blank and save | Validation error; task not saved | Negative |
| CRT-03 | Max length | Enter exactly the maximum allowed characters | Accepted and displayed correctly | Edge |
| CRT-04 | Over max length | Enter max + 1 characters | Error or truncation warning; no crash | Negative |
| CRT-05 | Special characters/emoji | Title with emoji or Unicode | Saved and displayed correctly | Edge |
| CRT-06 | XSS/SQLi in task | `<script>` or `' OR 1=1` in fields | Escaped/sanitized; no execution | Security |
| CRT-07 | Past due date | Set due date to yesterday | Warning or rejection based on business rule | Edge |
| CRT-08 | Create while logged out | Try to create a task without logging in | Redirect/401; task not created | Negative |
| CRT-09 | DB/network failure | Simulate DB down while saving | Friendly error; no partial save | Negative |

### Task CRUD — View

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| VW-01 | View own tasks | Log in and open the task list | Only own tasks are shown | Positive |
| VW-02 | Empty state | New user with no tasks | "No tasks found" message shown | Edge |
| VW-03 | Multiple tasks | Create 10+ tasks | List loads; sorting/pagination works | Positive |
| VW-04 | Another user's task | Change task ID in the URL | 403/Not found; no data leak | Negative |
| VW-05 | Long title | Create a task with a very long title | Truncate/tooltip/scroll; layout not broken | Edge |
| VW-06 | Refresh persistence | Refresh the task list page | Tasks remain from the database | Positive |
| VW-07 | Deleted task | Delete a task and refresh | Deleted task is not shown | Positive |

### Task CRUD — Edit

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| EDT-01 | Edit own task | Change title/date/priority and save | Changes saved and visible | Positive |
| EDT-02 | Invalid edit | Empty title or invalid date | Validation error; no update | Negative |
| EDT-03 | Non-existing task | Edit a non-existent task ID | Task not found error | Negative |
| EDT-04 | Edit another user's task | Change the ID and try to edit | 403/Not found; no update | Negative |
| EDT-05 | Cancel edit | Open edit, make changes, cancel | No changes saved | Positive |
| EDT-06 | Concurrent edit | Two sessions edit the same task and save | Conflict detected or last-save warning | Edge |
| EDT-07 | Persistence | Edit, save, refresh | Updated data remains | Positive |

### Task CRUD — Delete

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| DEL-01 | Delete own task | Click delete and confirm | Task removed from list and database | Positive |
| DEL-02 | Cancel delete | Click delete and cancel | Task remains unchanged | Positive |
| DEL-03 | Non-existing task | Delete an already deleted or invalid ID | Task not found error | Negative |
| DEL-04 | Delete another user's task | Change the ID and try to delete | 403/Not found; no deletion | Negative |
| DEL-05 | Delete all tasks | Delete all tasks one by one | Empty state shown | Edge |
| DEL-06 | DB failure on delete | Simulate DB down during delete | Error shown; no partial delete | Negative |
| DEL-07 | Undo/soft delete | If supported, undo a delete | Task restored correctly | Edge |

### Input Validation & Error Handling

| ID | Test Scenario | Steps / Data | Expected Result | Type |
|---|---|---|---|---|
| VAL-01 | Required fields | Submit with mandatory fields empty | Field-specific required errors | Negative |
| VAL-02 | Email format | Enter an invalid email | Clear format error | Negative |
| VAL-03 | Password policy | Short, common, or no special character | Policy error with rules | Negative |
| VAL-04 | Max length | Enter very long text in all fields | Prevented or handled without crash | Edge |
| VAL-05 | Date format | Enter `32/13/2025` | Validation error | Negative |
| VAL-06 | Priority/status values | Send invalid values via UI/API | Only allowed values accepted | Negative |
| VAL-07 | Error messages | Trigger validation errors | Messages clear, specific, non-technical | Positive |
| VAL-08 | Server/DB error | Stop DB or simulate 500 | Friendly error; no stack trace shown | Negative |
| VAL-09 | Session expired | Expire session and perform an action | Redirect to login; action not saved | Edge |
| VAL-10 | Network offline | Disable network and submit | Offline/retry message; no data corruption | Negative |
| VAL-11 | SQLi/XSS | Inject in all input fields | Escaped/sanitized; no execution | Security |
| VAL-12 | Unicode/emoji | Use Hindi/Chinese/emoji text | Saved and displayed correctly | Edge |

---

## 3. Bug / Risk Areas (without executing the application)

| Bug ID | Description | Severity | Reason / Impact |
|---|---|---|---|
| BUG-01 | Missing authorization on task APIs — changing the task ID exposes another user's task | Critical | IDOR/BOLA; data leak; unauthorized edit/delete |
| BUG-02 | Passwords stored in plain text or with weak hashing | Critical | Database leak compromises all accounts |
| BUG-03 | SQL Injection in login, registration, or task fields | Critical | Database read/modify/delete; login bypass |
| BUG-04 | Stored XSS in task title, description, or name | Critical | Script executes in another user's browser; session theft |
| BUG-05 | No lockout or rate limiting on login | Major | Brute-force and credential stuffing possible |
| BUG-06 | Weak password policy | Major | Guessable passwords allowed |
| BUG-07 | Weak session management — no timeout, insecure cookie, fixation | Major/Critical | Session hijacking; unauthorized access |
| BUG-08 | No server-side validation or length check; raw DB errors exposed | Major | Truncation, crash, DoS, information disclosure |
| BUG-09 | Lost update / last-write-wins on concurrent edit | Major | One user's change overwrites another's |
| BUG-10 | No confirmation, soft delete, or undo on delete | Minor | Accidental permanent data loss; poor UX |

---

