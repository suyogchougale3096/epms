-- Active: 1768725356242@@127.0.0.1@5432@postgres
# 🏢 Employee & Payroll Management System (EPMS)
## User Stories & Acceptance Criteria

> **Project:** Employee & Payroll Management System
> **Company Context:** TechSoft Pune — 300 Employees
> **Stack:** Angular (Frontend) + Spring Boot (Backend) + MySQL (Database)
> **Version:** 1.0

---

## 📌 Table of Contents

1. [Epic 1: Authentication & Security](#epic-1-authentication--security)
2. [Epic 2: Employee Onboarding](#epic-2-employee-onboarding)
3. [Epic 3: Attendance Management](#epic-3-attendance-management)
4. [Epic 4: Leave Management](#epic-4-leave-management)
5. [Epic 5: Payroll Engine](#epic-5-payroll-engine)
6. [Epic 6: Tax Calculation](#epic-6-tax-calculation)
7. [Epic 7: Payslip Generation](#epic-7-payslip-generation)
8. [Epic 8: Performance Review](#epic-8-performance-review)
9. [Epic 9: Recruitment Management](#epic-9-recruitment-management)
10. [Epic 10: Training & Development](#epic-10-training--development)
11. [Epic 11: Reports & Compliance](#epic-11-reports--compliance)
12. [Epic 12: Admin Dashboard](#epic-12-admin-dashboard)

---

## 🔐 Epic 1: Authentication & Security

### US-001 — Login (Admin / HR / Manager / Employee)

**As a** registered user,
**I want to** log into the system using my credentials,
**So that** I can securely access my role-specific dashboard.

**Real World Scenario:**
> HR Manager Sneha Joshi opens EPMS on Monday morning. She enters her company email `sneha.joshi@techsoft.in` and password. System recognizes her as `HR_MANAGER` and redirects her to the HR Dashboard showing today's attendance summary, pending leave requests, and upcoming appraisals.

#### ✅ Acceptance Criteria

**Frontend (Angular):**
- Login page must have: Company Email, Password fields with validation.
- Show/Hide password toggle must be present.
- "Forgot Password?" link visible.
- On success → redirect to role-based dashboard:
  - `ADMIN` → `/admin/dashboard`
  - `HR` → `/hr/dashboard`
  - `MANAGER` → `/manager/dashboard`
  - `EMPLOYEE` → `/employee/dashboard`
- On failure → show: *"Invalid credentials. Please try again."*
- After 5 failed attempts → show: *"Account locked. Contact HR."*
- Loader spinner during API call.
- Fully responsive (mobile + desktop).

**Backend (Spring Boot):**
- `POST /api/auth/login` → accepts `{ email, password }`.
- Password verified using BCrypt.
- Return JWT token with: `userId`, `role`, `employeeId`, `departmentId`, `exp`.
- JWT expiry: 8 hours.
- Account lock after 5 failed attempts for 30 minutes.
- HTTP Status: `200 OK`, `401 Unauthorized`, `423 Locked`.

**Database (MySQL):**
- `users` table: `id`, `email`, `password_hash`, `role`, `is_active`, `failed_attempts`, `locked_until`, `last_login`, `created_at`.
- Role ENUM: `ADMIN`, `HR`, `MANAGER`, `EMPLOYEE`.

**Security:**
- All endpoints except `/api/auth/**` require valid JWT.
- CORS configured for Angular origin only.
- HTTPS enforced in production.
- JWT stored in HttpOnly cookie (not localStorage) to prevent XSS.

---

### US-002 — Forgot Password / Reset via OTP

**As a** user who forgot their password,
**I want to** reset it via OTP sent to my company email,
**So that** I can regain access without contacting IT.

**Real World Scenario:**
> Developer Ravi comes back from 2-week leave and forgets his password. He clicks "Forgot Password", enters `ravi.kumar@techsoft.in`, receives a 6-digit OTP, and sets a new password in under 2 minutes.

#### ✅ Acceptance Criteria

**Frontend:**
- Email input → OTP verification screen → New password screen.
- OTP: 6-digit input fields, 5-minute countdown timer, "Resend OTP" after expiry.
- New password + Confirm password must match; show strength indicator.
- Success: *"Password reset successfully! Please login."*

**Backend:**
- `POST /api/auth/forgot-password` → generate & email OTP.
- `POST /api/auth/verify-otp` → validate OTP.
- `POST /api/auth/reset-password` → save new BCrypt password.
- OTP expires in 5 min; invalidated after use.
- Rate limit: 3 requests/hour per email.

**Database:**
- `password_reset_tokens`: `id`, `user_id (FK)`, `otp`, `expires_at`, `is_used`.

---

### US-003 — Role-Based Access Control (RBAC)

**As a** system,
**I want to** restrict features based on user roles,
**So that** each user sees only what is relevant to them.

#### ✅ Acceptance Criteria

| Feature | Admin | HR | Manager | Employee |
|---|---|---|---|---|
| Onboard Employee | ✅ | ✅ | ❌ | ❌ |
| View All Employees | ✅ | ✅ | ✅ (own team) | ❌ |
| Approve Leave | ✅ | ✅ | ✅ (own team) | ❌ |
| Run Payroll | ✅ | ✅ | ❌ | ❌ |
| View Own Payslip | ✅ | ✅ | ✅ | ✅ |
| Performance Review | ✅ | ✅ | ✅ (own team) | ✅ (view own) |
| View Reports | ✅ | ✅ | ✅ (own team) | ❌ |

**Frontend:**
- Angular `canActivate` Route Guards check JWT role before loading pages.
- Sidebar navigation dynamically renders based on role.
- Unauthorized access → redirect to `403 Forbidden` page.

**Backend:**
- Spring Security `@PreAuthorize("hasAnyRole('ADMIN','HR')")` on all controllers.
- `403 Forbidden` returned for unauthorized access.

---

## 👤 Epic 2: Employee Onboarding

### US-004 — HR Onboards New Employee

**As an** HR,
**I want to** register a new employee with all their details,
**So that** they get an Employee ID, system access, and are ready to work from Day 1.

**Real World Scenario:**
> TechSoft hires a new Java Developer, Amit Desai, joining on 1st July. HR Sneha opens EPMS, fills in Amit's personal info, uploads his Aadhaar + PAN + degree certificates, assigns him to "Engineering" department under Manager Vikram, sets his CTC as ₹8,00,000/year, and submits. System generates `EMP-2024-0301`, creates his login credentials, and sends a welcome email with login details and joining instructions.

#### ✅ Acceptance Criteria

**Frontend:**
- Multi-step onboarding form:
  - **Step 1 — Personal Info:** Full Name, DOB, Gender, Blood Group, Personal Email, Phone, Emergency Contact.
  - **Step 2 — Address:** Current Address, Permanent Address (copy checkbox).
  - **Step 3 — Job Info:** Department (dropdown), Designation, Manager (dropdown), Employment Type (Full-Time/Part-Time/Contract), Date of Joining, Work Location.
  - **Step 4 — Salary Info:** CTC, Basic %, HRA %, Special Allowance — system auto-calculates breakdown.
  - **Step 5 — Bank & Tax:** Bank Name, Account No, IFSC Code, PAN Number, Aadhaar Number.
  - **Step 6 — Document Upload:** Aadhaar, PAN, Degree Certificate, Experience Letter, Photo (each max 2MB, PDF/JPG/PNG).
- Progress bar across steps.
- Preview page before final submit.
- Auto-generate Employee ID on success: `EMP-YEAR-XXXXX`.
- Send welcome email with temporary password.

**Backend:**
- `POST /api/hr/employees` → create employee + user account.
- Auto-generate `employeeId` format: `EMP-2024-00301`.
- `GET /api/hr/employees` → paginated list with search & filters.
- `PUT /api/hr/employees/{id}` → update employee details.
- `DELETE /api/hr/employees/{id}` → soft delete (set `is_active = false`, set `exit_date`).
- File upload: `MultipartFile` stored in `/uploads/employees/{employeeId}/`.

**Database:**
- `employees`: `id`, `employee_id (unique)`, `user_id (FK)`, `full_name`, `dob`, `gender`, `blood_group`, `phone`, `emergency_contact`, `department_id (FK)`, `designation_id (FK)`, `manager_id (FK → employees)`, `employment_type`, `date_of_joining`, `exit_date`, `is_active`, `work_location`.
- `employee_bank_details`: `id`, `employee_id (FK)`, `bank_name`, `account_no`, `ifsc_code`.
- `employee_tax_info`: `id`, `employee_id (FK)`, `pan_number`, `aadhaar_number`, `tax_regime` (OLD/NEW).
- `employee_documents`: `id`, `employee_id (FK)`, `doc_type`, `file_url`, `uploaded_at`.
- `departments`: `id`, `name`, `head_id (FK → employees)`.
- `designations`: `id`, `title`, `level`, `department_id (FK)`.

**Security:**
- Only `ADMIN` and `HR` roles can onboard employees.
- PAN and Aadhaar stored encrypted (AES-256).
- Document files accessible only via signed URLs.

---

### US-005 — Employee Views & Updates Own Profile

**As an** Employee,
**I want to** view my profile and update personal details,
**So that** my records are always accurate.

**Real World Scenario:**
> Amit moves to a new apartment. He logs into EPMS, goes to "My Profile", clicks "Edit", updates his current address and emergency contact number, and saves. HR gets a notification: *"Amit Desai has updated profile. Pending review."*

#### ✅ Acceptance Criteria

**Frontend:**
- Profile page: Photo, Personal Info, Job Info (read-only), Bank Details (partially masked), Documents.
- Employee can edit: Address, Phone, Emergency Contact, Profile Photo, Bank Details.
- Job Info (Designation, Department, Manager, CTC) — read-only for employee.
- Changes go to HR for approval before being saved (workflow).
- Profile photo upload with crop feature.

**Backend:**
- `GET /api/employees/{id}` → get profile (employee can only get own; manager can get team's).
- `PUT /api/employees/{id}/profile` → submit profile update request.
- `PUT /api/hr/employees/{id}/approve-update` → HR approves/rejects update.

**Database:**
- `profile_update_requests`: `id`, `employee_id (FK)`, `field_name`, `old_value`, `new_value`, `status` (PENDING/APPROVED/REJECTED), `reviewed_by (FK)`, `requested_at`.

---

### US-006 — Department & Designation Management

**As an** Admin/HR,
**I want to** manage departments and designations,
**So that** employees can be correctly categorized.

#### ✅ Acceptance Criteria

**Frontend:**
- Department list: Name, Head, Total Employees — Add/Edit/Delete.
- Designation list: Title, Level, Department — Add/Edit/Delete.
- Org Chart view: visual tree of departments → managers → employees.

**Backend:**
- Full CRUD for `/api/departments` and `/api/designations`.
- Cannot delete department if employees are assigned to it.
- `GET /api/departments/{id}/org-chart` → returns hierarchical employee tree.

---

## 📅 Epic 3: Attendance Management

### US-007 — Biometric / Manual Attendance

**As an** HR/System,
**I want to** record employee attendance daily,
**So that** accurate working hours are maintained for payroll.

**Real World Scenario:**
> Every morning at TechSoft, employees punch in at the biometric machine. The machine's data syncs with EPMS every 15 minutes via API. For remote employees, they manually check-in from the EPMS app. By 9:30 AM, HR can see who's present, absent, late, or WFH.

#### ✅ Acceptance Criteria

**Frontend:**
- HR Attendance Dashboard: Today's summary — Present / Absent / Late / WFH / On Leave counts.
- Attendance table: Employee Name, Check-In, Check-Out, Hours Worked, Status.
- Filter by: Department, Date, Status.
- Manual attendance entry for HR (for system failures).
- Employee: "Check In / Check Out" button on dashboard (for manual/WFH).
- Monthly calendar view: color-coded attendance per day.

**Backend:**
- `POST /api/attendance/sync` → receive biometric data (batch import JSON).
- `POST /api/attendance/checkin` → manual check-in with timestamp + IP/location.
- `POST /api/attendance/checkout` → check-out.
- `GET /api/attendance/today` → today's summary (HR view).
- `GET /api/attendance/employee/{id}/monthly` → monthly report.
- Auto-calculate: Hours Worked = Check-Out − Check-In; Late if Check-In > 9:30 AM.
- If no check-out by midnight → flag as "Incomplete".

**Database:**
- `attendance`: `id`, `employee_id (FK)`, `date`, `check_in`, `check_out`, `hours_worked`, `status` (PRESENT/ABSENT/LATE/HALF_DAY/WFH/ON_LEAVE), `source` (BIOMETRIC/MANUAL/APP), `is_regularized`.
- `attendance_regularization`: `id`, `attendance_id (FK)`, `reason`, `approved_by (FK)`, `status`.

**Security:**
- Biometric sync endpoint secured with API Key (not JWT).
- Employees can only view their own attendance.

---

### US-008 — Attendance Regularization

**As an** Employee,
**I want to** request correction of wrong attendance entries,
**So that** my payroll is not incorrectly deducted.

**Real World Scenario:**
> Ravi worked from home on Tuesday but forgot to check in on the app. The system marked him absent. He submits a regularization request with reason "WFH — forgot to check in". Manager Vikram approves it, and the status changes to WFH.

#### ✅ Acceptance Criteria

**Frontend:**
- Employee sees "Regularize" button next to incorrect attendance entries.
- Form: Date, Actual Check-In, Actual Check-Out, Reason, Supporting Doc (optional).
- Manager sees pending regularization requests with Approve/Reject.

**Backend:**
- `POST /api/attendance/regularize` → submit request.
- `PUT /api/attendance/regularize/{id}/approve` → manager approves, updates attendance.
- `PUT /api/attendance/regularize/{id}/reject` → reject with reason.

---

## 🏖️ Epic 4: Leave Management

### US-009 — Employee Applies for Leave

**As an** Employee,
**I want to** apply for leave online,
**So that** I don't need to fill paper forms and can track my application status.

**Real World Scenario:**
> Employee Priya wants to take 3 days of Casual Leave for Diwali. She opens EPMS, selects "Apply Leave", picks Casual Leave type, selects dates Nov 1–3, adds reason "Diwali holidays", and submits. Her manager Vikram gets an email + EPMS notification. He approves it. Priya gets notified, and her leave balance updates automatically.

#### ✅ Acceptance Criteria

**Frontend:**
- Leave Application form: Leave Type (dropdown), From Date, To Date, No. of Days (auto-calculated, skipping weekends/holidays), Reason, Handover Note, Attachment (optional).
- Leave Balance card showing: Earned Leave, Casual Leave, Sick Leave, Comp-Off — Available / Used / Balance.
- Leave history table: Type, Dates, Days, Status, Applied On.
- Cannot apply for leave if balance is 0.
- Cannot apply for past dates (except sick leave with medical certificate).

**Backend:**
- `POST /api/leaves/apply` → create leave request.
- `GET /api/leaves/employee/{id}/balance` → leave balance per type.
- `GET /api/leaves/employee/{id}/history` → leave history.
- Auto-calculate days: exclude weekends + public holidays from leave count.
- Validate: sufficient balance; no overlapping leave dates.

**Database:**
- `leave_types`: `id`, `name` (Casual/Sick/Earned/Comp-Off/Maternity/Paternity), `max_days_per_year`, `is_paid`, `carry_forward`.
- `leave_balances`: `id`, `employee_id (FK)`, `leave_type_id (FK)`, `year`, `allocated`, `used`, `balance`.
- `leave_requests`: `id`, `employee_id (FK)`, `leave_type_id (FK)`, `from_date`, `to_date`, `days`, `reason`, `status` (PENDING/APPROVED/REJECTED/CANCELLED), `approved_by (FK)`, `applied_at`.
- `public_holidays`: `id`, `date`, `name`, `year`.

---

### US-010 — Manager Approves / Rejects Leave

**As a** Manager,
**I want to** review and approve/reject leave requests from my team,
**So that** team availability is managed without disruption.

**Real World Scenario:**
> Manager Vikram opens his dashboard on Monday and sees 3 pending leave requests from his team. He sees Priya's 3-day Diwali leave, checks that no one else is on leave those days (team calendar view), and approves. For another request, he sees the project deadline clashes and rejects with comment: *"Critical deadline on Nov 2. Please reschedule."*

#### ✅ Acceptance Criteria

**Frontend:**
- Manager dashboard shows pending leave requests count.
- Leave approval page: Employee Name, Leave Type, Dates, Days, Reason, Leave Balance.
- Team Leave Calendar: visual calendar showing who is on leave on which dates.
- Approve/Reject with optional comments.
- Bulk approve option.

**Backend:**
- `GET /api/leaves/manager/{managerId}/pending` → list pending requests.
- `PUT /api/leaves/{id}/approve` → approve, deduct from leave balance, notify employee.
- `PUT /api/leaves/{id}/reject` → reject with reason, notify employee.
- `GET /api/leaves/team/{managerId}/calendar` → team leave calendar data.

---

## 💰 Epic 5: Payroll Engine

### US-011 — HR Configures Salary Structure

**As an** HR,
**I want to** configure salary components for each employee,
**So that** payroll is calculated correctly every month.

**Real World Scenario:**
> Amit Desai joined with CTC ₹8,00,000/year. HR creates his salary structure:
> Basic = 40% of CTC = ₹26,667/month
> HRA = 50% of Basic = ₹13,333/month
> Special Allowance = ₹16,667/month
> PF = 12% of Basic (₹3,200) deducted, company also contributes ₹3,200.
> Professional Tax = ₹200/month (Maharashtra slab).

#### ✅ Acceptance Criteria

**Frontend:**
- Salary structure form per employee: CTC input → system auto-calculates all components.
- Show breakdown: Earnings (Basic, HRA, Special Allowance, Other) vs Deductions (PF, PT, TDS, Loan EMI).
- Bulk salary revision: upload Excel to revise CTC for multiple employees.
- Effective date for salary changes.

**Backend:**
- `POST /api/payroll/salary-structure` → save salary structure.
- `GET /api/payroll/salary-structure/{employeeId}` → get current structure.
- `POST /api/payroll/salary-structure/bulk-import` → Excel bulk update.
- Auto-calculate components based on percentages.
- Maintain history: salary revisions logged with effective date.

**Database:**
- `salary_components`: `id`, `name` (Basic/HRA/Special Allowance/PF/PT/TDS), `type` (EARNING/DEDUCTION), `calculation_type` (FIXED/PERCENTAGE), `percentage_of`.
- `employee_salary_structure`: `id`, `employee_id (FK)`, `ctc`, `basic`, `hra`, `special_allowance`, `other_allowances`, `pf_employee`, `pf_employer`, `professional_tax`, `effective_from`, `effective_to`.

---

### US-012 — HR Runs Monthly Payroll

**As an** HR,
**I want to** process monthly payroll for all employees in one click,
**So that** salaries are calculated accurately and on time.

**Real World Scenario:**
> On 28th of every month, HR Sneha opens EPMS, clicks "Run Payroll → November 2024". System fetches attendance for all 300 employees, calculates present days, applies LOP (Loss of Pay) for unpaid absences, deducts PF/PT/TDS, and generates payroll summary. Sneha reviews, approves, and system sends salary to payroll bank file and payslips to all employees.

#### ✅ Acceptance Criteria

**Frontend:**
- "Run Payroll" button for selected month/year.
- Processing screen with progress bar (300 employees).
- Payroll summary table: Employee, Gross Salary, LOP Days, LOP Deduction, PF, PT, TDS, Net Pay.
- Filter: Department, Status (Processed/Pending/Error).
- Edit individual record before final approval.
- "Approve & Disburse" button → locks payroll and triggers payslip generation.
- Export payroll data as Excel / Bank Transfer File.

**Backend:**
- `POST /api/payroll/run` → `{ month, year }` → triggers payroll for all active employees.
- For each employee:
  1. Fetch attendance → calculate working days, LOP days.
  2. Apply LOP: `LOP Amount = (Basic + DA) / Working Days in Month × LOP Days`.
  3. Calculate Gross Pay = Sum of all earnings − LOP.
  4. Apply deductions: PF, PT, TDS.
  5. Net Pay = Gross − Total Deductions.
- `POST /api/payroll/approve/{month}/{year}` → lock and disburse.
- `GET /api/payroll/summary/{month}/{year}` → summary report.

**Database:**
- `payroll_runs`: `id`, `month`, `year`, `status` (DRAFT/APPROVED/DISBURSED), `run_by (FK)`, `run_at`, `approved_by (FK)`, `approved_at`.
- `payroll_records`: `id`, `payroll_run_id (FK)`, `employee_id (FK)`, `working_days`, `present_days`, `lop_days`, `gross_salary`, `lop_deduction`, `pf_employee`, `pf_employer`, `professional_tax`, `tds`, `other_deductions`, `net_pay`, `status`.

---

## 🧾 Epic 6: Tax Calculation

### US-013 — TDS Calculation (Old & New Tax Regime)

**As an** HR/System,
**I want to** calculate TDS accurately for each employee,
**So that** tax compliance is maintained and employees are not under/over-taxed.

**Real World Scenario:**
> Amit Desai chose New Tax Regime during onboarding. At year start, HR inputs his declared investments. System calculates annual taxable income, applies tax slabs, divides TDS equally across 12 months. In January, Amit submits actual investment proofs — system recalculates and adjusts remaining TDS.

#### ✅ Acceptance Criteria

**Frontend:**
- Tax Declaration form (April each year): HRA exemption, 80C investments (PPF, LIC, ELSS), 80D (Health Insurance), Home Loan, Other deductions.
- TDS preview: Annual Income → Deductions → Taxable Income → Tax → Monthly TDS.
- Tax Regime selector: Old vs New — show comparison of tax in both.
- Investment Proof submission (Jan–Feb): Upload documents for each declared investment.

**Backend:**
- `POST /api/tax/declaration/{employeeId}` → save investment declaration.
- `GET /api/tax/tds-preview/{employeeId}` → calculate and return TDS projection.
- Tax slab computation for both Old and New regime (FY 2024-25 slabs).
- `POST /api/tax/proof-submission/{employeeId}` → save proofs, recalculate TDS.
- Auto-update monthly TDS in payroll after recalculation.

**Database:**
- `tax_declarations`: `id`, `employee_id (FK)`, `financial_year`, `regime`, `hra_claimed`, `80c_amount`, `80d_amount`, `home_loan_interest`, `other_deductions`, `declared_at`.
- `investment_proofs`: `id`, `declaration_id (FK)`, `category`, `amount`, `file_url`, `verified_by (FK)`.

---

### US-014 — Professional Tax Management

**As an** HR/System,
**I want to** auto-deduct Professional Tax based on state and salary slab,
**So that** PT compliance is maintained for all employees.

#### ✅ Acceptance Criteria

**Backend:**
- Maharashtra PT slabs auto-applied (e.g., ₹200/month for salary > ₹10,000).
- PT deducted monthly in payroll.
- PT Challan generated monthly for government submission.

**Database:**
- `professional_tax_slabs`: `id`, `state`, `min_salary`, `max_salary`, `monthly_pt`.

---

## 📄 Epic 7: Payslip Generation

### US-015 — Auto-Generate & Email Payslips

**As a** System,
**I want to** generate PDF payslips and email them to employees,
**So that** every employee has a record of their monthly salary.

**Real World Scenario:**
> On 1st December, after payroll approval, all 300 employees of TechSoft receive an email: *"Your November 2024 payslip is ready."* Amit opens it — a professional PDF shows his earnings, deductions, net pay, and YTD figures. He downloads it for his home loan application.

#### ✅ Acceptance Criteria

**Frontend:**
- Employee: "My Payslips" page → list of monthly payslips → click to view/download PDF.
- Payslip PDF format:
  - Company logo, name, address.
  - Employee: Name, ID, Designation, Department, PAN, Bank Account (masked).
  - Earnings table: Basic, HRA, Special Allowance, Other Allowances, Gross Earnings.
  - Deductions table: PF (Employee + Employer), PT, TDS, Loan EMI, Total Deductions.
  - Net Pay (highlighted).
  - YTD (Year-to-Date) figures.
  - Month/Year, working days, paid days.
- HR: Bulk email all payslips with one click.
- Password-protected PDF: password = `EMPID + DOB (DDMMYYYY)`.

**Backend:**
- `POST /api/payslips/generate/{month}/{year}` → generate all payslips after payroll approval.
- `GET /api/payslips/{employeeId}/{month}/{year}` → get specific payslip.
- PDF generated using iText or JasperReports.
- Email sent via SMTP/SendGrid with PDF attachment.
- `POST /api/payslips/email-all/{month}/{year}` → bulk email.

**Database:**
- `payslips`: `id`, `employee_id (FK)`, `payroll_record_id (FK)`, `month`, `year`, `pdf_url`, `emailed_at`, `generated_at`.

---

## ⭐ Epic 8: Performance Review

### US-016 — Manager Conducts Performance Review

**As a** Manager,
**I want to** review and rate my team members' performance quarterly/annually,
**So that** appraisals and increments are based on objective data.

**Real World Scenario:**
> TechSoft conducts Annual Appraisals in March. Manager Vikram logs into EPMS, goes to "Performance Reviews → Annual 2024". He sees his 8 team members. For Amit, he rates: Technical Skills (4/5), Communication (3/5), Delivery (5/5), Teamwork (4/5). He adds comments and submits. Amit gets notified and can view his review. HR uses this data for increment calculation.

#### ✅ Acceptance Criteria

**Frontend:**
- HR creates Review Cycle: Name (Q1/Annual), Period, Deadline, Rating Scale (1–5).
- Manager sees team members to review with completion status.
- Review form: Competency ratings (customizable), Goals Achievement, Strengths, Areas of Improvement, Overall Rating, Manager Comments.
- Employee Self-Assessment form: submit before manager review.
- After manager submits → employee views review and can add acknowledgement comment.
- Rating summary chart per employee.

**Backend:**
- `POST /api/reviews/cycles` → HR creates review cycle.
- `POST /api/reviews/self-assessment` → employee self-assessment.
- `POST /api/reviews/manager-review` → manager submits review.
- `GET /api/reviews/employee/{id}/history` → all reviews for an employee.
- `GET /api/reviews/cycle/{id}/summary` → HR sees all review statuses.

**Database:**
- `review_cycles`: `id`, `name`, `type` (QUARTERLY/ANNUAL), `start_date`, `end_date`, `status`.
- `performance_reviews`: `id`, `cycle_id (FK)`, `employee_id (FK)`, `reviewer_id (FK)`, `self_rating`, `manager_rating`, `technical_score`, `communication_score`, `delivery_score`, `teamwork_score`, `overall_rating`, `comments`, `status` (PENDING/SUBMITTED/ACKNOWLEDGED).
- `review_goals`: `id`, `review_id (FK)`, `goal`, `achievement_percent`, `comments`.

---

### US-017 — Increment & Promotion Based on Review

**As an** HR,
**I want to** process increments and promotions based on performance ratings,
**So that** deserving employees are rewarded and salary data is updated.

**Real World Scenario:**
> After Annual Review 2024, HR Sneha runs the Increment Process. She sees Amit got Overall Rating 4.2/5. HR applies 15% increment — CTC changes from ₹8L to ₹9.2L effective April 1st. Amit's designation upgrades from "Software Engineer" to "Senior Software Engineer". Revised payslips generate from April.

#### ✅ Acceptance Criteria

**Frontend:**
- Increment sheet: Employee, Current CTC, Rating, Suggested Increment %, New CTC — editable.
- Bulk increment via Excel upload.
- Promotion form: New Designation, Effective Date.
- Confirmation dialog before saving (increments are irreversible).

**Backend:**
- `POST /api/hr/increments` → save increment, update salary structure with new effective date.
- `POST /api/hr/promotions` → update designation, maintain history.
- Previous salary structure retained (for historical payslips).

---

## 🧑‍💼 Epic 9: Recruitment Management

### US-018 — HR Posts Job Opening

**As an** HR,
**I want to** post job openings and manage the hiring pipeline,
**So that** TechSoft can hire the right talent efficiently.

**Real World Scenario:**
> TechSoft needs 5 Java Developers. HR Sneha creates a Job Opening: "Java Developer - 2+ years - Pune - ₹5–10L". Posts it internally (for referrals) and externally. Applications start coming in — HR screens resumes, schedules interviews, and finally offers the role to selected candidates.

#### ✅ Acceptance Criteria

**Frontend:**
- Job Opening form: Title, Department, Positions, Skills, Experience, Location, Salary Range, Description, Deadline.
- Jobs list with Active/Closed/On-Hold status.
- Application tracking board (Kanban style): Applied → Screening → Interview → Offer → Hired/Rejected.
- Resume viewer (PDF inline viewer).
- Schedule Interview: Date, Time, Interviewer (multi-select), Mode (In-Person/Video).
- Send offer letter via email.

**Backend:**
- `POST /api/recruitment/jobs` → create job opening.
- `GET /api/recruitment/jobs` → list with filters.
- `POST /api/recruitment/applications` → candidate applies.
- `PUT /api/recruitment/applications/{id}/status` → move through pipeline.
- `POST /api/recruitment/interviews/schedule` → schedule interview, send calendar invites.
- `POST /api/recruitment/offer` → generate offer letter PDF and email.
- On Hire → auto-trigger onboarding (pre-fill US-004 form).

**Database:**
- `job_openings`: `id`, `title`, `department_id (FK)`, `positions`, `skills`, `experience_min`, `salary_min`, `salary_max`, `status`, `posted_by (FK)`, `deadline`.
- `candidates`: `id`, `name`, `email`, `phone`, `resume_url`, `source` (Portal/Referral/Direct).
- `applications`: `id`, `job_id (FK)`, `candidate_id (FK)`, `status`, `applied_at`.
- `interviews`: `id`, `application_id (FK)`, `scheduled_at`, `mode`, `interviewers (JSON)`, `feedback`, `result`.

---

## 📚 Epic 10: Training & Development

### US-019 — HR Manages Training Programs

**As an** HR,
**I want to** create and assign training programs to employees,
**So that** employees continuously upskill.

**Real World Scenario:**
> TechSoft introduces Spring Boot 3.0. HR creates a training: "Spring Boot 3.0 Upgrade — 3 days — All Java Developers". Assigns it to 45 employees. Trainer uploads materials and recordings. After training, employees take an online quiz. Completion certificates auto-generated.

#### ✅ Acceptance Criteria

**Frontend:**
- Training form: Title, Description, Trainer, Type (Internal/External/Online), Duration, Skills, Assigned Employees.
- Training calendar: upcoming programs.
- Employee: My Trainings → view materials, mark attendance, take quiz.
- Certificate download on 100% completion.
- Training feedback form after completion.

**Backend:**
- `POST /api/training/programs` → create training.
- `POST /api/training/programs/{id}/assign` → assign employees.
- `POST /api/training/attendance` → mark attendance per session.
- `POST /api/training/quiz/submit` → submit quiz, auto-grade.
- `POST /api/training/certificate/generate/{employeeId}/{programId}` → PDF certificate.

**Database:**
- `training_programs`: `id`, `title`, `trainer`, `type`, `start_date`, `end_date`, `skills`.
- `training_assignments`: `id`, `program_id (FK)`, `employee_id (FK)`, `completion_status`, `quiz_score`, `certificate_url`.
- `training_materials`: `id`, `program_id (FK)`, `title`, `file_url`, `type` (PDF/VIDEO/LINK).

---

## 📊 Epic 11: Reports & Compliance

### US-020 — HR Generates Statutory Reports

**As an** HR,
**I want to** generate compliance reports (PF, PT, Form 16),
**So that** TechSoft meets all government statutory requirements.

**Real World Scenario:**
> March 31st deadline approaches. HR needs to submit PF ECR (Electronic Challan cum Return) for March, Professional Tax Challan for Maharashtra, and generate Form 16 for all employees for FY 2024-25. EPMS generates all these with one click.

#### ✅ Acceptance Criteria

**Frontend:**
- Reports menu: Statutory Reports, Payroll Reports, Attendance Reports, Leave Reports, Headcount Reports.
- Each report: select date range → generate → Preview → Download (PDF/Excel).
- Schedule auto-reports: e.g., "Email monthly attendance report to HR every 1st".

**Backend:**
- `GET /api/reports/pf-ecr/{month}/{year}` → PF ECR file in government format.
- `GET /api/reports/pt-challan/{month}/{year}` → PT Challan PDF.
- `GET /api/reports/form16/{employeeId}/{financialYear}` → Form 16 PDF.
- `GET /api/reports/headcount` → department-wise employee count.
- `GET /api/reports/attrition/{year}` → monthly joining vs exit report.
- `GET /api/reports/leave-summary/{month}` → department-wise leave report.

**Database:**
- `report_schedules`: `id`, `report_type`, `frequency`, `recipients`, `last_run`, `next_run`.

---

## 🖥️ Epic 12: Admin Dashboard

### US-021 — HR/Admin Dashboard Overview

**As an** Admin/HR,
**I want to** see a real-time overview of the entire workforce,
**So that** I can monitor and act quickly on important metrics.

**Real World Scenario:**
> Monday 9 AM. HR Sneha opens her dashboard. She sees: 287 Present, 13 Absent (with names), 5 On Leave, 3 Regularization Pending, 8 Leave Approvals Pending, Payroll Status (20 days to next payroll), 2 Work Anniversaries today. She clicks on "Absents" and sends a WhatsApp reminder in one click.

#### ✅ Acceptance Criteria

**Frontend:**
- **KPI Cards:** Total Employees, Present Today, Absent Today, On Leave, New Joiners This Month, Exits This Month.
- **Attendance Chart:** Weekly present % trend (line chart).
- **Department Headcount:** Bar chart.
- **Leave Status:** Pie chart — Approved / Pending / Rejected this month.
- **Payroll Widget:** Days to next payroll, last payroll status.
- **Upcoming Events:** Birthdays (today/this week), Work Anniversaries, Training Sessions.
- **Pending Actions:** Leave approvals, Regularization requests, Profile update requests — clickable quick links.
- **Recent Activities Feed:** Last 10 system events.

**Backend:**
- `GET /api/dashboard/hr/stats` → single API returning all dashboard data.
- Aggregated using JPA + native SQL for performance.
- Cache stats for 10 minutes (Spring Cache + Redis optional).

**Security:**
- HR/Admin dashboard API → `ADMIN` or `HR` role only.
- Manager dashboard → filtered to own team only.
- Employee dashboard → own data only.

---

## 🗄️ Complete Database Schema Overview

```
CORE TABLES
────────────────────────────────────────────────────────────
users                    employees
─────────────            ─────────────────────────────────
id (PK)                  id (PK)
email                    employee_id (unique)
password_hash            user_id (FK → users)
role (ENUM)              department_id (FK)
is_active                designation_id (FK)
                         manager_id (FK → employees)
                         date_of_joining, exit_date

departments              designations
─────────────            ─────────────────────────────────
id (PK)                  id (PK)
name                     title, level
head_id (FK)             department_id (FK)

ATTENDANCE & LEAVE
────────────────────────────────────────────────────────────
attendance               leave_requests
─────────────            ─────────────────────────────────
id (PK)                  id (PK)
employee_id (FK)         employee_id (FK)
date                     leave_type_id (FK)
check_in, check_out      from_date, to_date
status (ENUM)            days, status (ENUM)
hours_worked             approved_by (FK)

leave_balances           public_holidays
─────────────            ─────────────────────────────────
employee_id (FK)         id (PK)
leave_type_id (FK)       date, name, year
allocated, used

PAYROLL & TAX
────────────────────────────────────────────────────────────
employee_salary_structure   payroll_records
─────────────────────────   ──────────────────────────────
id (PK)                     id (PK)
employee_id (FK)            payroll_run_id (FK)
ctc, basic, hra             employee_id (FK)
special_allowance           gross_salary, net_pay
pf_employee, tds            lop_days, lop_deduction
effective_from              pf, pt, tds deductions

payslips                 tax_declarations
─────────────            ─────────────────────────────────
id (PK)                  id (PK)
employee_id (FK)         employee_id (FK)
payroll_record_id (FK)   financial_year, regime
month, year              80c, 80d, hra_claimed
pdf_url                  total_deductions

PERFORMANCE & RECRUITMENT
────────────────────────────────────────────────────────────
performance_reviews      job_openings
─────────────────────    ─────────────────────────────────
id (PK)                  id (PK)
cycle_id (FK)            title, department_id (FK)
employee_id (FK)         positions, status
reviewer_id (FK)         salary_min, salary_max
overall_rating           deadline

applications             interviews
─────────────            ─────────────────────────────────
id (PK)                  id (PK)
job_id (FK)              application_id (FK)
candidate_id (FK)        scheduled_at, mode
status (ENUM)            result, feedback
```

---

## 🔐 Security Checklist (Overall)

- [ ] JWT Authentication — all protected routes
- [ ] BCrypt password hashing
- [ ] Role-Based Access Control (Spring Security + Angular Guards)
- [ ] PAN & Aadhaar stored AES-256 encrypted
- [ ] Input validation — Frontend + Backend both layers
- [ ] SQL Injection prevention via JPA Parameterized Queries
- [ ] XSS protection — Angular auto-escaping + CSP headers
- [ ] CORS configured for Angular origin only
- [ ] File upload validation (type + size + virus scan)
- [ ] Rate limiting on Auth APIs
- [ ] Account lockout after 5 failed attempts
- [ ] Password-protected payslip PDFs
- [ ] Biometric sync API secured with API Key
- [ ] Employees can ONLY access their own data (JWT userId validation)
- [ ] Sensitive fields (salary, bank) masked in list views
- [ ] All file downloads via signed/expiring URLs
- [ ] Audit logs for all sensitive actions (payroll run, salary change, etc.)
- [ ] HTTPS enforced in production

---

## 🚀 Development Sprints (Recommended Order)

```
Sprint 1  →  US-001, US-002, US-003        Auth + RBAC
Sprint 2  →  US-004, US-005, US-006        Employee Onboarding
Sprint 3  →  US-007, US-008                Attendance
Sprint 4  →  US-009, US-010                Leave Management
Sprint 5  →  US-011, US-012                Payroll Engine
Sprint 6  →  US-013, US-014                Tax & Professional Tax
Sprint 7  →  US-015                        Payslip Generation
Sprint 8  →  US-016, US-017                Performance Review
Sprint 9  →  US-018                        Recruitment
Sprint 10 →  US-019                        Training
Sprint 11 →  US-020                        Reports & Compliance
Sprint 12 →  US-021                        Dashboard & Polish
```

---

*Document Version: 1.0 | Employee & Payroll Management System — TechSoft Pune*
