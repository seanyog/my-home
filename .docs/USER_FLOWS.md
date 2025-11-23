# User Flows & User Journeys
## Family House Management Application

**Document Purpose:** Define user interactions, workflows, and journeys through the application to guide UX design and implementation.

---

## User Personas

### Primary: Sarah (The Family Manager)
- **Age:** 38
- **Role:** Stay-at-home parent, manages household
- **Tech Savviness:** Medium
- **Pain Points:** Overwhelmed with tracking everything, mental load, spouse unaware of family schedule
- **Goals:** Centralize information, delegate tasks, reduce stress
- **Devices:** Primarily desktop/tablet, occasional mobile

### Secondary: Michael (The Working Partner)
- **Age:** 40
- **Role:** Works full-time, participates in family management
- **Tech Savviness:** Medium-High
- **Pain Points:** Out of the loop, unclear on responsibilities, doesn't know financial status
- **Goals:** Quick updates on family status, clear task lists, financial transparency
- **Devices:** Primarily mobile, occasional desktop

### Tertiary: Emma (Teen Child)
- **Age:** 16
- **Role:** High school student, driver's license
- **Tech Savviness:** High
- **Pain Points:** Unaware of family schedule, doesn't know if car is available
- **Goals:** Know family schedule, understand expectations, coordinate with parents
- **Devices:** Mobile only

---

## Core User Flows

### 1. Onboarding & Family Setup

#### Flow: New User Registration

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Landing Page                                             │
│    - Value proposition                                      │
│    - "Get Started" CTA                                      │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Sign Up Form                                             │
│    - Email                                                  │
│    - Password (with strength indicator)                     │
│    - Family name                                            │
│    - Timezone selection                                     │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Email Verification                                       │
│    - "Check your email" message                             │
│    - Resend verification link                               │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼ (user clicks email link)
┌─────────────────────────────────────────────────────────────┐
│ 4. Welcome & Profile Setup                                  │
│    - Your name                                              │
│    - Your role (dropdown: Parent/Guardian/Other)            │
│    - Upload photo (optional)                                │
│    - Notification preferences                               │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Family Setup Wizard                                      │
│    ┌──────────────────────────────────────────────────┐    │
│    │ Step 1: Add Family Members                       │    │
│    │ - Invite spouse/partner (email)                  │    │
│    │ - Add children (name, age, role)                 │    │
│    │ - Skip for now option                            │    │
│    └──────────────────────────────────────────────────┘    │
│                                                             │
│    ┌──────────────────────────────────────────────────┐    │
│    │ Step 2: What do you want to track? (multi-select)│   │
│    │ ☑ Family Calendar                                │    │
│    │ ☑ Bills & Finances                               │    │
│    │ ☑ Vehicles                                       │    │
│    │ ☑ Tasks & Shopping Lists                         │    │
│    │ ☐ Documents                                      │    │
│    └──────────────────────────────────────────────────┘    │
│                                                             │
│    ┌──────────────────────────────────────────────────┐    │
│    │ Step 3: Quick Start Options                      │    │
│    │ - Import Google Calendar                         │    │
│    │ - Link Bank Account                              │    │
│    │ - Add First Vehicle                              │    │
│    │ - Or "Do this later"                             │    │
│    └──────────────────────────────────────────────────┘    │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Dashboard (First Time)                                   │
│    - Welcome message                                        │
│    - Quick action cards                                     │
│    - Tutorial tooltips                                      │
│    - "Take a tour" option                                   │
└─────────────────────────────────────────────────────────────┘
```

**Success Criteria:**
- User completes registration in < 5 minutes
- At least one family member invited
- At least one module configured

**Error Handling:**
- Email already exists → Offer login or password reset
- Weak password → Show requirements and suggestions
- Verification email not received → Resend option
- Network error → Save progress, retry

---

### 2. Calendar Management

#### Flow: Creating a Family Event

**User Story:** *As Sarah, I want to create a soccer practice event that repeats weekly, so my family knows when to take Emma to practice.*

```
Dashboard → Calendar Tab
  │
  ▼
Calendar View (Month/Week/Day toggle)
  │
  ▼
Click "New Event" button or Click time slot
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Create Event Modal                                          │
│                                                             │
│ Event Title: [Soccer Practice                    ]         │
│ Category:    [Activities ▼]                                │
│              (School, Activities, Medical, Celebration,     │
│               Work, Other)                                  │
│                                                             │
│ Date & Time:                                                │
│   Start: [Dec 1, 2025] [2:00 PM]                           │
│   End:   [Dec 1, 2025] [3:30 PM]                           │
│   ☐ All day event                                          │
│                                                             │
│ Location:   [Community Soccer Field              ]         │
│             (autocomplete from previous locations)          │
│                                                             │
│ Who's attending:                                            │
│   ☑ Emma                                                   │
│   ☑ Sarah (You)                                            │
│   ☐ Michael                                                │
│                                                             │
│ Repeat: [Weekly ▼]                                         │
│   Every: [1] week(s) on [Monday ▼]                         │
│   Ends:  [● After 10 occurrences                           │
│           ○ On date                                        │
│           ○ Never                                          │
│                                                             │
│ Reminders:                                                  │
│   ☑ 1 day before (Email to attending members)              │
│   ☑ 1 hour before (Push notification)                      │
│   + Add another reminder                                   │
│                                                             │
│ Notes: [Bring water bottle and shin guards      ]         │
│        (optional)                                           │
│                                                             │
│ [Cancel]  [Save Event]                                     │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Event Created → Calendar updated with recurring events
  │
  ▼
Success notification: "Soccer Practice added to calendar (10 occurrences)"
  │
  ▼
Email sent to attending members (Emma, Sarah)
Mobile push notification to Emma
```

**Alternative Flow: Import from Google Calendar**

```
Calendar Tab → Settings icon → Integrations
  │
  ▼
Connect Google Calendar button
  │
  ▼
Google OAuth consent screen
  │
  ▼ (user grants permission)
Sync in progress... (background job)
  │
  ▼
"20 events imported from Google Calendar"
  │
  ▼
Calendar view shows imported events (different color/icon)
```

**Edge Cases:**
- **Conflict Detection:** "Michael has a work meeting at this time. Continue anyway?"
- **Past Date:** Warning: "This event is in the past. Are you sure?"
- **Missing Required Field:** Highlight field in red, show error message
- **Sync Failure:** "Unable to sync with Google Calendar. Try again?"

---

### 3. Financial Account Linking

#### Flow: Linking Bank Account via Plaid

**User Story:** *As Michael, I want to link my checking account so I can see my balance alongside my other accounts.*

```
Dashboard → Finances Tab → Accounts
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Financial Accounts Overview                                 │
│                                                             │
│ Total Balance: $12,450.00                                   │
│                                                             │
│ Accounts:                                                   │
│   [Chase Checking]        $3,200.00                        │
│   [Ally Savings]          $8,500.00                        │
│   [Capital One Credit]   -$750.00                          │
│                                                             │
│ [+ Link New Account]                                       │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "Link New Account")
┌─────────────────────────────────────────────────────────────┐
│ Link Financial Account                                      │
│                                                             │
│ We use Plaid to securely connect to your bank.             │
│ Your credentials are never stored by us.                    │
│                                                             │
│ [Continue with Plaid]    [Enter Manually Instead]         │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "Continue with Plaid")
┌─────────────────────────────────────────────────────────────┐
│ Plaid Link Modal                                            │
│                                                             │
│ Search for your bank: [_________________] 🔍               │
│                                                             │
│ Popular Banks:                                              │
│   [Chase]  [Bank of America]  [Wells Fargo]                │
│   [Citi]   [Capital One]      [US Bank]                    │
│                                                             │
│ Or scroll to find your institution...                       │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (user selects "Wells Fargo")
┌─────────────────────────────────────────────────────────────┐
│ Wells Fargo Login (in Plaid iframe)                         │
│                                                             │
│ Username: [________________]                                │
│ Password: [________________]                                │
│                                                             │
│ [Login]                                                     │
│                                                             │
│ 🔒 Secure connection via Plaid                             │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (successful login)
┌─────────────────────────────────────────────────────────────┐
│ Select Accounts to Link                                     │
│                                                             │
│ ☑ Wells Fargo Checking (...1234)    $4,582.19             │
│ ☑ Wells Fargo Savings (...5678)     $12,340.00            │
│ ☐ Wells Fargo Credit (...9012)      $-1,250.00            │
│                                                             │
│ [Cancel]  [Link Selected Accounts]                         │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "Link Selected Accounts")
Processing... (exchanging token, fetching accounts)
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Success! Accounts Linked                                    │
│                                                             │
│ ✓ Wells Fargo Checking                                     │
│ ✓ Wells Fargo Savings                                      │
│                                                             │
│ Syncing transactions... (may take a few minutes)           │
│                                                             │
│ [View Accounts]                                            │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Financial Accounts page updated with new accounts
Background job syncs transactions
```

**Background Process:**
```
1. Exchange public token for access token (server)
2. Store encrypted access token
3. Fetch account details (balances, metadata)
4. Store accounts in database
5. Queue transaction sync job
6. Fetch transactions (last 90 days)
7. Categorize transactions
8. Update account balance
9. Send notification: "2 new accounts linked, 143 transactions imported"
```

**Error Scenarios:**
- **Invalid Credentials:** "Login failed. Please check username/password."
- **MFA Required:** Plaid handles (SMS/email code input)
- **Institution Down:** "Wells Fargo is currently unavailable. Try again later."
- **Rate Limited:** "Too many attempts. Please wait 5 minutes and try again."

---

### 4. Bill Tracking & Reminders

#### Flow: Adding a Recurring Bill

**User Story:** *As Sarah, I want to add our electric bill so I'm reminded to pay it before the due date.*

```
Finances → Bills Tab
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Bills & Subscriptions                                       │
│                                                             │
│ Upcoming Bills:                                             │
│   ⚠️ Electric Bill - Due Dec 15 ($150.00)  [Mark Paid]    │
│   Internet - Due Dec 20 ($80.00)                           │
│   Car Insurance - Due Dec 25 ($125.00)                     │
│                                                             │
│ Monthly Total: $355.00                                      │
│                                                             │
│ [+ Add New Bill]                                           │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "Add New Bill")
┌─────────────────────────────────────────────────────────────┐
│ Add New Bill                                                │
│                                                             │
│ Bill Name:   [Electric Bill - PG&E              ]          │
│                                                             │
│ Category:    [Utilities ▼]                                 │
│              (Utilities, Insurance, Subscription,           │
│               Mortgage, Loan, Other)                        │
│                                                             │
│ Amount:      [$] [150.00]                                  │
│              ☑ Amount varies (will remind to enter)        │
│                                                             │
│ Due Date:    [15th] of each month                          │
│                                                             │
│ Recurring:   ☑ Yes                                         │
│   Frequency: [Monthly ▼]                                   │
│                                                             │
│ Payment Method (optional):                                  │
│   [Wells Fargo Checking ▼]                                 │
│   ☑ Auto-pay enabled                                       │
│                                                             │
│ Reminders:                                                  │
│   ☑ 7 days before (Email)                                  │
│   ☑ 3 days before (Email + Push)                           │
│   ☑ 1 day before (Email + Push)                            │
│   ☐ On due date (SMS) - Premium feature                    │
│                                                             │
│ Notes:      [Account #123456789                 ]          │
│             (optional)                                      │
│                                                             │
│ [Cancel]  [Save Bill]                                      │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Bill saved → Appears in Bills list
  │
  ▼
Success: "Electric Bill added. Next due: Dec 15, 2025"
```

**Reminder Flow (7 days before due):**

```
Background Job (runs daily at 9 AM user's timezone)
  │
  ▼
Check for bills due in 7 days
  │
  ▼
Electric Bill found (due Dec 15, today is Dec 8)
  │
  ▼
Send Email:
┌─────────────────────────────────────────────────────────────┐
│ Subject: Bill Reminder: Electric Bill due in 7 days         │
│                                                             │
│ Hi Sarah,                                                   │
│                                                             │
│ Your Electric Bill - PG&E is due on December 15, 2025.     │
│ Amount: $150.00                                             │
│                                                             │
│ Payment Account: Wells Fargo Checking                       │
│ Auto-pay: Enabled ✓                                        │
│                                                             │
│ [View Bill Details] [Mark as Paid]                         │
│                                                             │
│ This is an automated reminder from Family House Management. │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Send Push Notification (if enabled):
"💰 Electric Bill due in 7 days ($150.00)"
```

**Marking Bill as Paid:**

```
User clicks "Mark Paid" in email or app
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Mark Bill as Paid                                           │
│                                                             │
│ Electric Bill - PG&E                                        │
│ Amount: $150.00                                             │
│                                                             │
│ Paid on:     [Dec 12, 2025 ▼]                              │
│ Paid from:   [Wells Fargo Checking ▼]                      │
│                                                             │
│ Actual amount (if different): [$] [152.45]                 │
│                                                             │
│ Notes:       [Included late fee $2.45       ]              │
│              (optional)                                     │
│                                                             │
│ [Cancel]  [Confirm Payment]                                │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Bill marked as paid → Status updated
  │
  ▼
Next occurrence scheduled (Jan 15, 2026)
Success: "Electric Bill marked as paid. Next due: Jan 15, 2026"
```

---

### 5. Vehicle Maintenance Tracking

#### Flow: Adding a Vehicle & Logging Maintenance

**User Story:** *As Michael, I want to add our family car and log the recent oil change so I know when the next service is due.*

```
Dashboard → Vehicles Tab
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ My Vehicles                                                 │
│                                                             │
│ No vehicles added yet.                                      │
│                                                             │
│ [+ Add Your First Vehicle]                                 │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "Add Your First Vehicle")
┌─────────────────────────────────────────────────────────────┐
│ Add Vehicle                                                 │
│                                                             │
│ Vehicle Name:   [Family SUV                    ]           │
│                 (nickname for easy identification)          │
│                                                             │
│ Make:           [Toyota ▼]                                 │
│ Model:          [Highlander              ]                 │
│ Year:           [2022]                                     │
│                                                             │
│ VIN (optional): [1HGBH41JXMN109186]                        │
│ License Plate:  [ABC1234]                                  │
│ Color:          [Blue]                                     │
│                                                             │
│ Current Mileage: [25,000] miles                            │
│                                                             │
│ Purchase Info (optional):                                   │
│   Date:  [Mar 15, 2022]                                    │
│   Price: [$] [45,000]                                      │
│                                                             │
│ Insurance:                                                  │
│   Provider:      [State Farm              ]                │
│   Policy Number: [SF-123456789            ]                │
│   Renewal Date:  [Mar 15, 2026]                            │
│   Annual Cost:   [$] [1,800]                               │
│                                                             │
│ Registration:                                               │
│   Expiration:    [Mar 31, 2026]                            │
│   Renewal Cost:  [$] [120]                                 │
│                                                             │
│ [Cancel]  [Save Vehicle]                                   │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Vehicle saved → Vehicle card appears
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Family SUV - 2022 Toyota Highlander                         │
│                                                             │
│ 🚗 25,000 miles                                             │
│                                                             │
│ Upcoming:                                                   │
│   ⚠️ Insurance Renewal - Mar 15, 2026 (112 days)           │
│   ⚠️ Registration Renewal - Mar 31, 2026 (128 days)        │
│                                                             │
│ [View Details] [Add Maintenance] [Edit]                    │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (user clicks "Add Maintenance")
┌─────────────────────────────────────────────────────────────┐
│ Log Maintenance Record                                      │
│                                                             │
│ Service Type: [Oil Change ▼]                               │
│               (Oil Change, Tire Rotation, Inspection,       │
│                Brake Service, Repair, Other)                │
│                                                             │
│ Date:         [Nov 20, 2025]                               │
│ Mileage:      [24,500] miles                               │
│                                                             │
│ Cost:         [$] [75.00]                                  │
│                                                             │
│ Service Provider:                                           │
│   Name:    [Quick Lube Express            ]                │
│   Phone:   [(555) 123-4567]                                │
│   Address: [123 Main St, Anytown, CA 12345]                │
│                                                             │
│ Notes:       [Full synthetic oil, new oil filter]          │
│              (optional)                                     │
│                                                             │
│ Receipt: [Upload File] or [Take Photo]                     │
│          (optional)                                         │
│                                                             │
│ Next Service Due:                                           │
│   ☑ Automatically calculate (3,000 miles or 3 months)      │
│   Date:    [Feb 20, 2026] (estimated)                      │
│   Mileage: [27,500] miles                                  │
│                                                             │
│ [Cancel]  [Save Record]                                    │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Maintenance record saved
Current mileage updated to 24,500
Next oil change reminder scheduled
  │
  ▼
Success: "Maintenance record added. Next oil change: Feb 20, 2026"
```

**Reminder Flow (1 week before next service):**

```
Background Job (weekly check)
  │
  ▼
Check vehicles for upcoming maintenance
  │
  ▼
Family SUV: Oil change due Feb 20 (today is Feb 13)
  │
  ▼
Send notification to all family admins (Sarah, Michael):
"🚗 Family SUV: Oil change due in 1 week (Feb 20 or 27,500 miles)"
  │
  ▼
Display in Dashboard under "Upcoming Tasks"
```

**Cost Analytics View:**

```
Vehicle Details → Costs Tab
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Family SUV - Annual Cost Breakdown                          │
│                                                             │
│ Total Cost (2025): $2,450.00                                │
│                                                             │
│ ┌─────────────────────────────────────────────────────┐    │
│ │     Insurance      Maintenance    Registration      │    │
│ │      $1,800           $500            $120          │    │
│ │     (73%)           (20%)            (5%)           │    │
│ └─────────────────────────────────────────────────────┘    │
│                                                             │
│ Maintenance History (2025):                                 │
│   Nov 20: Oil Change - $75.00                              │
│   Aug 15: Tire Rotation - $50.00                           │
│   May 10: Annual Inspection - $85.00                       │
│   Feb 05: Oil Change - $75.00                              │
│                                                             │
│ Cost per Mile: $0.098                                       │
│ (based on 25,000 miles in 2025)                            │
│                                                             │
│ [Export Report]  [View All Records]                        │
└─────────────────────────────────────────────────────────────┘
```

---

### 6. Task Management & Delegation

#### Flow: Creating and Assigning a Task

**User Story:** *As Sarah, I want to create a task for Michael to pick up groceries on his way home.*

```
Dashboard → Tasks Tab
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Family Tasks                                                │
│                                                             │
│ Filter: [All Tasks ▼] [All Members ▼] [Sort: Due Date ▼]  │
│                                                             │
│ To Do (3):                                                  │
│   ☐ Pick up dry cleaning - Michael - Today                 │
│   ☐ Schedule dentist appointment - Sarah - Dec 2           │
│   ☐ Buy Emma's birthday gift - Michael - Dec 10            │
│                                                             │
│ In Progress (1):                                            │
│   ☐ Plan holiday party - Sarah - Dec 20                    │
│                                                             │
│ Completed (5):                                              │
│   ☑ Grocery shopping - Michael - Completed Nov 21          │
│   ...                                                       │
│                                                             │
│ [+ New Task]                                               │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (click "+ New Task")
┌─────────────────────────────────────────────────────────────┐
│ Create New Task                                             │
│                                                             │
│ Task Title: [Pick up groceries                  ]          │
│                                                             │
│ Description (optional):                                     │
│ ┌──────────────────────────────────────────────────────┐  │
│ │ Get items from shopping list:                        │  │
│ │ - Milk (whole)                                       │  │
│ │ - Bread                                              │  │
│ │ - Chicken breast (2 lbs)                            │  │
│ │ - Fresh vegetables                                   │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ Assign to:   [Michael ▼]                                   │
│              (Sarah, Michael, Emma, Unassigned)             │
│                                                             │
│ Due Date:    [Today, 6:00 PM]                              │
│ Priority:    [Medium ▼] (Low, Medium, High, Urgent)        │
│                                                             │
│ Linked Shopping List:                                       │
│   [Weekly Groceries ▼]                                     │
│   (creates checklist from shopping list)                    │
│                                                             │
│ Reminder:                                                   │
│   ☑ Notify assignee when created                           │
│   ☑ Remind 1 hour before due time                          │
│                                                             │
│ [Cancel]  [Create Task]                                    │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Task created and assigned to Michael
  │
  ▼
Michael receives notification:
  Email: "Sarah assigned you a task: Pick up groceries (Due: Today 6 PM)"
  Push: "📋 New task: Pick up groceries - Due today 6 PM"
```

**Michael's Experience (Mobile):**

```
Michael receives push notification at 2 PM
  │
  ▼
Taps notification → Opens app to task details
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Pick up groceries                                           │
│                                                             │
│ Assigned by: Sarah                                          │
│ Due: Today, 6:00 PM (⏰ 4 hours left)                       │
│ Priority: Medium                                            │
│                                                             │
│ Description:                                                │
│ Get items from shopping list:                               │
│ ☐ Milk (whole)                                             │
│ ☐ Bread                                                    │
│ ☐ Chicken breast (2 lbs)                                   │
│ ☐ Fresh vegetables                                         │
│                                                             │
│ [Start Task] [Reassign] [Change Due Date]                  │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (clicks "Start Task")
Status changed to "In Progress"
Sarah receives notification: "Michael started working on: Pick up groceries"
  │
  ▼
Michael checks off items as he shops
  │
  ▼ (5:45 PM - all items checked)
┌─────────────────────────────────────────────────────────────┐
│ All items checked!                                          │
│                                                             │
│ ✓ Milk (whole)                                             │
│ ✓ Bread                                                    │
│ ✓ Chicken breast (2 lbs)                                   │
│ ✓ Fresh vegetables                                         │
│                                                             │
│ [Mark Task Complete]                                       │
└─────────────────────────────────────────────────────────────┘
  │
  ▼ (clicks "Mark Task Complete")
Task marked as completed
  │
  ▼
Sarah receives notification: "✅ Michael completed: Pick up groceries"
Task moves to "Completed" section with timestamp
```

**Alternative: Recurring Task Template**

```
Tasks Tab → Templates → Create Template
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Create Task Template                                        │
│                                                             │
│ Template Name: [Weekly Grocery Shopping         ]          │
│                                                             │
│ Task Title:    [Grocery shopping                ]          │
│ Description:   [Get items from weekly groceries ]          │
│                list                               ]          │
│                                                             │
│ Assign to:     [Alternate between Michael & Sarah]         │
│ Priority:      [Medium ▼]                                  │
│                                                             │
│ Recurrence:                                                 │
│   Frequency: [Weekly ▼]                                    │
│   Day:       [Saturday ▼]                                  │
│   Time:      [4:00 PM]                                     │
│                                                             │
│ Auto-create:                                                │
│   ☑ Create task 1 day in advance                           │
│   ☑ Notify assignee when created                           │
│                                                             │
│ [Cancel]  [Save Template]                                  │
└─────────────────────────────────────────────────────────────┘
  │
  ▼
Template saved → Tasks auto-created every week
```

---

### 7. Dashboard Overview

#### User Experience: Daily Check-In

**Sarah's Morning Routine:**

```
Sarah opens app at 8:00 AM
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ Good morning, Sarah! ☀️                                     │
│                                                             │
│ ┌────────────────────────── Today ──────────────────────┐  │
│ │ 📅 3 events                                            │  │
│ │   9:00 AM - Emma's School Field Trip (all day)        │  │
│ │   2:00 PM - Soccer Practice (Emma, Sarah)             │  │
│ │   6:30 PM - Dinner with Friends (All)                 │  │
│ │                                                        │  │
│ │ ✓ 2 tasks due                                          │  │
│ │   ☐ Schedule dentist appointment (Sarah)              │  │
│ │   ☐ Confirm dinner reservation (Michael)              │  │
│ └────────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─────────────────── This Week ───────────────────────┐    │
│ │ 💰 2 bills due                                         │  │
│ │   ⚠️ Electric Bill - Dec 15 ($150) [Mark Paid]       │  │
│ │   Internet - Dec 20 ($80)                             │  │
│ │                                                        │  │
│ │ 🚗 Vehicle reminder                                    │  │
│ │   Family SUV - Oil change due in 8 days               │  │
│ └────────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌────────────────── Financial Summary ─────────────────┐   │
│ │ 💵 Total Balance: $12,450.00                          │  │
│ │    ↑ Up $320 from last week                           │  │
│ │                                                        │  │
│ │ November Spending: $3,240 / $4,000 budget             │  │
│ │ ███████████████░░░░░ 81%                              │  │
│ │                                                        │  │
│ │ Top Categories:                                        │  │
│ │   Groceries: $680 | Utilities: $450 | Gas: $280      │  │
│ │                                                        │  │
│ │ [View Details]                                        │  │
│ └────────────────────────────────────────────────────────┘  │
│                                                             │
│ Quick Actions:                                              │
│ [+ New Event] [+ Add Bill] [+ Create Task] [+ Expense]     │
└─────────────────────────────────────────────────────────────┘
```

**Customizable Widgets:**
- User can add/remove/reorder dashboard widgets
- Available widgets: Calendar, Tasks, Bills, Financial Summary, Vehicle Reminders, Shopping Lists, Recent Activity
- Drag-and-drop interface for customization

---

## Mobile-Specific Flows

### Quick Actions (Mobile App)

**Home Screen Shortcuts:**
- Long-press app icon → Quick actions
  - Create Event
  - Add Expense
  - View Today's Schedule
  - Shopping List

**Widget Options:**
- Today's Events widget (small)
- Week at a Glance (medium)
- Tasks Due Today (small)
- Financial Summary (medium)

---

## Error States & Edge Cases

### Network Errors

```
User action → API call fails
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ ⚠️ Connection Error                                         │
│                                                             │
│ Unable to save changes. Please check your internet          │
│ connection and try again.                                   │
│                                                             │
│ [Retry]  [Save Offline]                                    │
└─────────────────────────────────────────────────────────────┘
```

**Offline Mode (PWA):**
- View cached data (read-only)
- Queue changes locally
- Sync when connection restored
- Show "offline" indicator in header

### Data Conflicts

```
User edits event → Save → Another user already modified
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│ ⚠️ Conflict Detected                                        │
│                                                             │
│ This event was modified by Michael while you were editing.  │
│                                                             │
│ Your changes:                                               │
│   Time: 2:00 PM - 3:30 PM                                  │
│                                                             │
│ Michael's changes:                                          │
│   Time: 2:30 PM - 4:00 PM                                  │
│                                                             │
│ [Keep Mine]  [Use Michael's]  [Merge Changes]             │
└─────────────────────────────────────────────────────────────┘
```

---

## Accessibility Considerations

### Keyboard Navigation
- All actions accessible via keyboard
- Tab order logical
- Escape key closes modals
- Keyboard shortcuts for common actions

### Screen Reader Support
- ARIA labels on all interactive elements
- Form validation errors announced
- Loading states announced
- Success/error messages announced

### Visual Accessibility
- Color contrast meets WCAG AA standards
- Don't rely on color alone (use icons + text)
- Resizable text (up to 200%)
- Focus indicators visible

---

## Conclusion

These user flows provide a comprehensive blueprint for implementing the Family House Management application. They:

1. **Prioritize user experience** with clear, intuitive workflows
2. **Handle edge cases** and error scenarios gracefully
3. **Support multiple user types** (admins, members, teens)
4. **Enable mobile-first** interactions
5. **Ensure accessibility** for all users
6. **Provide feedback** at every step

The flows should be used to:
- Guide frontend development
- Design API contracts
- Create test scenarios
- Inform UX design decisions
- Train customer support

As the product evolves, these flows will be refined based on real user feedback and usage patterns.
