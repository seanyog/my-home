# MVP Feature Breakdown & User Stories
## Family House Management Application

**Version:** 1.0
**Target Timeline:** 6 months
**Strategy:** Monolithic (Strategy 1)

---

## MVP Scope Definition

### What's IN the MVP ✅

**Core Philosophy:** Minimum features needed to provide value to a family managing their household.

1. **User Authentication & Family Management**
2. **Family Calendar** (basic CRUD + Google Calendar sync)
3. **Financial Account Linking** (Plaid integration, read-only)
4. **Bill Tracking** (manual entry + reminders)
5. **Basic Vehicle Management** (1 vehicle, maintenance log)
6. **Task System** (create, assign, complete)
7. **Dashboard** (unified view of calendar, bills, tasks)

### What's OUT of the MVP ❌

**Will be added in Phase 2+ based on user feedback:**

- Multiple vehicle support (limit: 1 vehicle in MVP)
- Document storage and OCR
- Budget planning and expense categorization
- Shopping lists
- Advanced analytics and reports
- Mobile native apps (PWA only)
- Calendar conflict detection
- Recurring task templates
- Credit card tracking (only bank accounts in MVP)
- Investment account tracking
- Multi-language support
- Advanced notification preferences
- Household maintenance scheduling
- Receipt scanning

---

## Epic Breakdown

### Epic 1: Foundation & Authentication
**Timeline:** Weeks 1-3
**Team:** Full team (3 developers)
**Priority:** P0 (Blocker)

#### User Stories

**1.1 User Registration**
```
As a new user,
I want to create an account with email and password,
So that I can start managing my family's information.

Acceptance Criteria:
- [ ] User can enter email, password, and family name
- [ ] Password must meet security requirements (8+ chars, uppercase, lowercase, number)
- [ ] Email verification sent upon registration
- [ ] User cannot log in until email is verified
- [ ] Duplicate email shows clear error message
- [ ] Family timezone is auto-detected or manually selectable

Technical Tasks:
- [ ] Create User and Family database tables (Prisma schema)
- [ ] Implement bcrypt password hashing
- [ ] Set up SendGrid for email verification
- [ ] Create JWT token generation/verification
- [ ] Build registration API endpoint
- [ ] Build registration UI (React form)

Estimate: 8 points (2 days)
```

**1.2 User Login**
```
As a registered user,
I want to log in with my email and password,
So that I can access my family's dashboard.

Acceptance Criteria:
- [ ] User can enter email and password
- [ ] Successful login redirects to dashboard
- [ ] Invalid credentials show clear error
- [ ] "Remember me" option keeps user logged in
- [ ] "Forgot password" link available
- [ ] Session expires after 7 days of inactivity

Technical Tasks:
- [ ] Create login API endpoint
- [ ] Implement JWT refresh token strategy
- [ ] Build login UI
- [ ] Set up HTTP-only cookie for refresh token
- [ ] Add "Remember me" logic
- [ ] Create password reset flow

Estimate: 5 points (1 day)
```

**1.3 Family Member Management**
```
As a family admin,
I want to invite other family members to join,
So that we can all access and update family information.

Acceptance Criteria:
- [ ] Admin can send invitation via email
- [ ] Invitation contains unique signup link
- [ ] Invitee can create account via link (auto-joins family)
- [ ] Invitee assigned appropriate role (Adult, Teen, Child)
- [ ] Admin can view list of all family members
- [ ] Admin can remove family members
- [ ] Invitations expire after 7 days

Technical Tasks:
- [ ] Create invitation token generation
- [ ] Build invitation email template
- [ ] Create invitation acceptance flow
- [ ] Build family members management UI
- [ ] Implement role-based permissions

Estimate: 8 points (2 days)
```

**Epic 1 Total Estimate:** 21 points (~5 days, Week 1-3 with setup)

---

### Epic 2: Calendar Management
**Timeline:** Weeks 4-6
**Team:** 2 developers
**Priority:** P0 (Must Have)

#### User Stories

**2.1 Create Calendar Event**
```
As a family member,
I want to create a calendar event,
So that my family knows about upcoming activities.

Acceptance Criteria:
- [ ] User can create event with title, date, time
- [ ] Optional fields: description, location, category
- [ ] User can select which family members are attending
- [ ] User can set event reminders (email/push)
- [ ] Event appears on family calendar immediately
- [ ] All attendees receive notification of new event

Technical Tasks:
- [ ] Create CalendarEvent database table
- [ ] Build event creation API endpoint
- [ ] Implement event validation (start < end time)
- [ ] Build event creation form UI
- [ ] Integrate FullCalendar.js library
- [ ] Set up event reminder job queue

Estimate: 13 points (3 days)
```

**2.2 View Calendar**
```
As a family member,
I want to view the family calendar in month/week/day views,
So that I can see what's scheduled.

Acceptance Criteria:
- [ ] Calendar shows all family events
- [ ] User can toggle between month/week/day views
- [ ] Events color-coded by category
- [ ] Click event to see details modal
- [ ] User can filter by family member
- [ ] Today's date highlighted

Technical Tasks:
- [ ] Build calendar view API endpoint (date range query)
- [ ] Integrate FullCalendar.js configuration
- [ ] Build event details modal
- [ ] Implement category color coding
- [ ] Add filter functionality

Estimate: 8 points (2 days)
```

**2.3 Edit/Delete Calendar Event**
```
As an event creator,
I want to edit or delete events I created,
So that I can keep the calendar up to date.

Acceptance Criteria:
- [ ] User can edit their own events
- [ ] Admins can edit any event
- [ ] Changes notify all attendees
- [ ] User can delete events with confirmation
- [ ] Deleted events removed from calendar
- [ ] Attendees notified of cancellation

Technical Tasks:
- [ ] Build update/delete API endpoints
- [ ] Implement permission checks (creator or admin)
- [ ] Build edit form (pre-populated)
- [ ] Add delete confirmation dialog
- [ ] Send update/cancellation notifications

Estimate: 5 points (1 day)
```

**2.4 Recurring Events**
```
As a family member,
I want to create recurring events (weekly soccer practice),
So that I don't have to create the same event repeatedly.

Acceptance Criteria:
- [ ] User can set recurrence pattern (daily/weekly/monthly)
- [ ] User can specify end date or number of occurrences
- [ ] All occurrences appear on calendar
- [ ] User can edit single occurrence or entire series
- [ ] User can delete single occurrence or entire series

Technical Tasks:
- [ ] Design recurrence data model (JSON field)
- [ ] Implement recurrence expansion logic
- [ ] Build recurrence UI (frequency selector)
- [ ] Add edit series vs single occurrence logic
- [ ] Update calendar query to handle recurrence

Estimate: 13 points (3 days)
```

**2.5 Google Calendar Sync**
```
As a family member,
I want to sync my Google Calendar,
So that all my events appear in one place.

Acceptance Criteria:
- [ ] User can connect Google Calendar via OAuth
- [ ] Initial sync imports last 90 days of events
- [ ] Events sync every 15 minutes
- [ ] Google events marked with icon/badge
- [ ] User can disconnect Google Calendar
- [ ] Sync errors shown with retry option

Technical Tasks:
- [ ] Set up Google OAuth 2.0 credentials
- [ ] Build Google Calendar API integration
- [ ] Create sync background job (BullMQ)
- [ ] Store OAuth tokens securely
- [ ] Build connection UI flow
- [ ] Handle sync errors gracefully

Estimate: 13 points (3 days)
```

**Epic 2 Total Estimate:** 52 points (~12 days, Weeks 4-6)

---

### Epic 3: Financial Integration
**Timeline:** Weeks 7-9
**Team:** 2 developers
**Priority:** P0 (Must Have)

#### User Stories

**3.1 Link Bank Account (Plaid)**
```
As a family admin,
I want to link my bank account via Plaid,
So that I can see my account balances.

Acceptance Criteria:
- [ ] User clicks "Link Account" button
- [ ] Plaid Link modal opens
- [ ] User selects bank and logs in
- [ ] User selects which accounts to link
- [ ] Accounts appear in dashboard with current balance
- [ ] Initial transactions synced (last 90 days)

Technical Tasks:
- [ ] Set up Plaid account (sandbox + development)
- [ ] Implement Plaid Link token creation
- [ ] Build token exchange endpoint
- [ ] Store access token encrypted
- [ ] Create FinancialAccount table
- [ ] Build Plaid Link UI integration
- [ ] Create transaction sync job

Estimate: 21 points (5 days)
```

**3.2 View Account Balances**
```
As a family member,
I want to see all linked account balances in one place,
So that I know our financial status at a glance.

Acceptance Criteria:
- [ ] Dashboard shows total balance across all accounts
- [ ] Individual account cards show name, type, balance
- [ ] Balances update when synced (max 6 hours old)
- [ ] Last sync timestamp shown
- [ ] Manual refresh button available
- [ ] Sync errors shown with troubleshooting

Technical Tasks:
- [ ] Build accounts list API endpoint
- [ ] Create account balance summary component
- [ ] Build manual sync trigger
- [ ] Display last sync timestamp
- [ ] Handle sync errors (expired token, bank down)

Estimate: 8 points (2 days)
```

**3.3 View Transactions**
```
As a family member,
I want to see recent transactions from linked accounts,
So that I can track our spending.

Acceptance Criteria:
- [ ] User can view transactions per account
- [ ] Transactions sorted by date (newest first)
- [ ] Shows date, description, amount, category
- [ ] User can filter by date range
- [ ] Transactions paginated (20 per page)
- [ ] Auto-syncs new transactions daily

Technical Tasks:
- [ ] Create Transaction table
- [ ] Build transaction list API with pagination
- [ ] Create daily sync job (BullMQ)
- [ ] Build transaction list UI
- [ ] Add date range filter
- [ ] Implement pagination

Estimate: 13 points (3 days)
```

**3.4 Account Management**
```
As a family admin,
I want to remove linked accounts,
So that I can manage which accounts are tracked.

Acceptance Criteria:
- [ ] User can unlink account with confirmation
- [ ] Unlinked accounts removed from dashboard
- [ ] Historical data preserved (for now)
- [ ] User can re-link same account later

Technical Tasks:
- [ ] Build account deletion API
- [ ] Add confirmation dialog
- [ ] Soft delete vs hard delete decision
- [ ] Build unlink UI

Estimate: 5 points (1 day)
```

**Epic 3 Total Estimate:** 47 points (~11 days, Weeks 7-9)

---

### Epic 4: Bill Tracking
**Timeline:** Weeks 10-11
**Team:** 1-2 developers
**Priority:** P1 (High Value)

#### User Stories

**4.1 Add Bill**
```
As a family admin,
I want to add a bill (e.g., electric bill),
So that I'm reminded to pay it on time.

Acceptance Criteria:
- [ ] User enters bill name, amount, due date
- [ ] User can mark as recurring (monthly/annually)
- [ ] User can select category (utilities, insurance, etc.)
- [ ] User can link to payment account (optional)
- [ ] Bill appears in bills list
- [ ] Automatic reminders scheduled

Technical Tasks:
- [ ] Create Bill table
- [ ] Build bill creation API
- [ ] Create reminder scheduling logic
- [ ] Build bill form UI
- [ ] Implement recurrence pattern

Estimate: 8 points (2 days)
```

**4.2 View Bills & Reminders**
```
As a family member,
I want to see upcoming bills,
So that I know what payments are due.

Acceptance Criteria:
- [ ] Dashboard shows bills due in next 30 days
- [ ] Bills sorted by due date
- [ ] Overdue bills highlighted in red
- [ ] Shows bill name, amount, due date, status
- [ ] User can click to see bill details

Technical Tasks:
- [ ] Build bills list API (filtered by date range)
- [ ] Create bills dashboard widget
- [ ] Build bill details modal
- [ ] Add status badges (pending/paid/overdue)

Estimate: 5 points (1 day)
```

**4.3 Bill Reminders**
```
As a family member,
I want to receive reminders before bills are due,
So that I don't forget to pay them.

Acceptance Criteria:
- [ ] Email sent 7 days before due date
- [ ] Email sent 3 days before due date
- [ ] Email sent 1 day before due date
- [ ] Reminders include bill details and payment link
- [ ] User can mark bill as paid from email

Technical Tasks:
- [ ] Create daily reminder check job
- [ ] Build email templates (SendGrid)
- [ ] Implement reminder delivery logic
- [ ] Add "mark as paid" link with token
- [ ] Log reminder history

Estimate: 8 points (2 days)
```

**4.4 Mark Bill as Paid**
```
As a family member,
I want to mark a bill as paid,
So that the system knows it's been handled.

Acceptance Criteria:
- [ ] User can mark bill as paid from UI or email
- [ ] User enters actual paid amount and date
- [ ] Bill status changes to "Paid"
- [ ] Paid bill moves to history
- [ ] Next occurrence scheduled (if recurring)

Technical Tasks:
- [ ] Build mark-as-paid API endpoint
- [ ] Update bill status logic
- [ ] Create next recurrence logic
- [ ] Build mark-as-paid UI
- [ ] Add payment history view

Estimate: 5 points (1 day)
```

**Epic 4 Total Estimate:** 26 points (~6 days, Weeks 10-11)

---

### Epic 5: Vehicle Management
**Timeline:** Weeks 12-13
**Team:** 1 developer
**Priority:** P1 (High Value)

#### User Stories

**5.1 Add Vehicle**
```
As a family admin,
I want to add our family vehicle,
So that I can track maintenance and costs.

Acceptance Criteria:
- [ ] User enters make, model, year, mileage
- [ ] Optional: VIN, license plate, purchase info
- [ ] Optional: insurance and registration details
- [ ] Vehicle card appears on Vehicles page
- [ ] Limited to 1 vehicle in MVP

Technical Tasks:
- [ ] Create Vehicle table
- [ ] Build vehicle creation API
- [ ] Build vehicle form UI
- [ ] Create vehicle details card component
- [ ] Add 1-vehicle limit validation

Estimate: 5 points (1 day)
```

**5.2 Log Maintenance**
```
As a family member,
I want to log vehicle maintenance (oil change),
So that I know when the next service is due.

Acceptance Criteria:
- [ ] User enters service type, date, mileage, cost
- [ ] Optional: service provider, notes, receipt
- [ ] User can set next service due (date/mileage)
- [ ] Maintenance record appears in history
- [ ] Current mileage updated

Technical Tasks:
- [ ] Create MaintenanceRecord table
- [ ] Build maintenance log API
- [ ] Build maintenance form UI
- [ ] Create maintenance history list
- [ ] Add next service calculation

Estimate: 8 points (2 days)
```

**5.3 Maintenance Reminders**
```
As a family member,
I want reminders when vehicle maintenance is due,
So that I don't miss important services.

Acceptance Criteria:
- [ ] Email reminder 1 week before due date
- [ ] Email reminder when mileage within 500 miles of due
- [ ] Dashboard shows upcoming maintenance
- [ ] Reminder includes service details

Technical Tasks:
- [ ] Create weekly maintenance check job
- [ ] Build reminder email template
- [ ] Add maintenance widget to dashboard
- [ ] Implement date and mileage-based triggers

Estimate: 5 points (1 day)
```

**5.4 Vehicle Cost Summary**
```
As a family member,
I want to see total vehicle costs,
So that I understand our vehicle expenses.

Acceptance Criteria:
- [ ] Shows annual cost breakdown (maintenance, insurance, registration)
- [ ] Shows cost per mile
- [ ] Shows maintenance history with costs
- [ ] Can filter by year

Technical Tasks:
- [ ] Build cost calculation logic
- [ ] Create cost summary API
- [ ] Build cost visualization (chart)
- [ ] Add year filter

Estimate: 5 points (1 day)
```

**Epic 5 Total Estimate:** 23 points (~5 days, Weeks 12-13)

---

### Epic 6: Task Management
**Timeline:** Weeks 14-15
**Team:** 1 developer
**Priority:** P1 (High Value)

#### User Stories

**6.1 Create Task**
```
As a family member,
I want to create a task (e.g., "Pick up groceries"),
So that it gets done.

Acceptance Criteria:
- [ ] User enters task title and description
- [ ] User can assign to family member
- [ ] User can set due date and priority
- [ ] Task appears in task list
- [ ] Assignee receives notification

Technical Tasks:
- [ ] Create Task table
- [ ] Build task creation API
- [ ] Build task form UI
- [ ] Send assignment notification
- [ ] Create task list component

Estimate: 5 points (1 day)
```

**6.2 Task List & Filters**
```
As a family member,
I want to see tasks filtered by status and assignee,
So that I know what needs to be done.

Acceptance Criteria:
- [ ] Tasks grouped by status (To Do, In Progress, Completed)
- [ ] User can filter by assignee
- [ ] User can sort by due date or priority
- [ ] Overdue tasks highlighted
- [ ] Shows task count per status

Technical Tasks:
- [ ] Build task list API with filters
- [ ] Create task board UI (Kanban-style)
- [ ] Add filter dropdowns
- [ ] Implement sorting logic
- [ ] Add overdue badge

Estimate: 8 points (2 days)
```

**6.3 Complete Task**
```
As an assignee,
I want to mark a task as complete,
So that everyone knows it's done.

Acceptance Criteria:
- [ ] User can mark task as complete
- [ ] Task moves to Completed section
- [ ] Task creator receives notification
- [ ] Completion timestamp recorded

Technical Tasks:
- [ ] Build task update API
- [ ] Add complete button/checkbox
- [ ] Send completion notification
- [ ] Update task status logic

Estimate: 3 points (0.5 day)
```

**6.4 Task Reminders**
```
As an assignee,
I want reminders for tasks due soon,
So that I don't forget them.

Acceptance Criteria:
- [ ] Email reminder 1 day before due date
- [ ] Email reminder on due date
- [ ] Dashboard shows tasks due today

Technical Tasks:
- [ ] Create daily task reminder job
- [ ] Build reminder email template
- [ ] Add tasks widget to dashboard
- [ ] Implement reminder logic

Estimate: 5 points (1 day)
```

**Epic 6 Total Estimate:** 21 points (~4.5 days, Weeks 14-15)

---

### Epic 7: Dashboard & Polish
**Timeline:** Weeks 16-18
**Team:** 2 developers
**Priority:** P0 (Must Have)

#### User Stories

**7.1 Unified Dashboard**
```
As a family member,
I want to see today's events, upcoming bills, and tasks in one place,
So that I know what's important.

Acceptance Criteria:
- [ ] Dashboard shows today's calendar events
- [ ] Dashboard shows bills due this week
- [ ] Dashboard shows tasks due today
- [ ] Dashboard shows total account balance
- [ ] Dashboard shows upcoming vehicle maintenance
- [ ] Widgets are responsive and mobile-friendly

Technical Tasks:
- [ ] Create dashboard API (aggregates data)
- [ ] Build dashboard layout (grid)
- [ ] Create widget components
- [ ] Add loading states
- [ ] Implement responsive design

Estimate: 13 points (3 days)
```

**7.2 Notification System**
```
As a family member,
I want to receive email notifications for important events,
So that I stay informed.

Acceptance Criteria:
- [ ] User receives email for: new event assigned, bill due soon, task assigned, maintenance due
- [ ] Emails use branded templates
- [ ] User can click links in email to view details
- [ ] Emails sent at appropriate times (9 AM user's timezone)

Technical Tasks:
- [ ] Create email templates (SendGrid)
- [ ] Build notification dispatch system
- [ ] Add timezone handling
- [ ] Test email deliverability
- [ ] Add unsubscribe links

Estimate: 8 points (2 days)
```

**7.3 User Profile & Settings**
```
As a user,
I want to update my profile and notification preferences,
So that I control my experience.

Acceptance Criteria:
- [ ] User can update name, email, photo
- [ ] User can change password
- [ ] User can set notification preferences
- [ ] User can set timezone
- [ ] Changes saved and reflected immediately

Technical Tasks:
- [ ] Build user update API
- [ ] Create profile settings page
- [ ] Build notification preferences UI
- [ ] Add photo upload (S3)
- [ ] Implement password change with validation

Estimate: 8 points (2 days)
```

**7.4 Mobile Responsiveness**
```
As a mobile user,
I want the app to work well on my phone,
So that I can use it anywhere.

Acceptance Criteria:
- [ ] All pages responsive (phone, tablet, desktop)
- [ ] Navigation works on mobile (hamburger menu)
- [ ] Forms easy to fill on mobile
- [ ] Tables/lists scroll horizontally if needed
- [ ] Touch targets sized appropriately

Technical Tasks:
- [ ] Audit all pages for mobile UX
- [ ] Implement mobile navigation
- [ ] Fix responsive issues
- [ ] Test on multiple devices
- [ ] Add PWA manifest

Estimate: 13 points (3 days)
```

**7.5 Error Handling & Loading States**
```
As a user,
I want clear feedback when things go wrong,
So that I know what to do.

Acceptance Criteria:
- [ ] Loading spinners shown during API calls
- [ ] Error messages clear and actionable
- [ ] Network errors handled gracefully
- [ ] 404 page for invalid routes
- [ ] 500 page for server errors
- [ ] Toast notifications for success/error

Technical Tasks:
- [ ] Create error boundary components
- [ ] Build toast notification system
- [ ] Add loading states to all async actions
- [ ] Create error and 404 pages
- [ ] Implement retry logic for failed requests

Estimate: 8 points (2 days)
```

**Epic 7 Total Estimate:** 50 points (~12 days, Weeks 16-18)

---

## Sprint Plan (6 Months = 24 Weeks)

### Phase 1: Foundation (Weeks 1-3)
- **Epic 1:** Authentication & Family Management
- **Deliverable:** Users can register, login, invite family members
- **Team:** 3 developers (full team onboarding)

### Phase 2: Calendar (Weeks 4-6)
- **Epic 2:** Calendar Management
- **Deliverable:** Full calendar functionality with Google sync
- **Team:** 2 developers

### Phase 3: Finances (Weeks 7-9)
- **Epic 3:** Financial Integration
- **Deliverable:** Link bank accounts, view balances and transactions
- **Team:** 2 developers

### Phase 4: Bills & Vehicles (Weeks 10-13)
- **Epic 4:** Bill Tracking (Weeks 10-11)
- **Epic 5:** Vehicle Management (Weeks 12-13)
- **Deliverable:** Bill reminders and vehicle maintenance tracking
- **Team:** 2 developers (can be split)

### Phase 5: Tasks & Dashboard (Weeks 14-18)
- **Epic 6:** Task Management (Weeks 14-15)
- **Epic 7:** Dashboard & Polish (Weeks 16-18)
- **Deliverable:** Unified dashboard with task system
- **Team:** 2 developers

### Phase 6: Testing & Launch Prep (Weeks 19-24)
- **Focus:** Bug fixes, testing, performance optimization, beta testing
- **Deliverables:**
  - Week 19-20: Internal testing, bug fixes
  - Week 21-22: Beta user testing (20-30 families)
  - Week 23: Final polish based on feedback
  - Week 24: Production deployment & soft launch
- **Team:** Full team (3 developers + QA)

---

## Story Point Estimation Guide

**1 point** = Half a day (~4 hours)
- Simple UI change, basic CRUD endpoint

**3 points** = 1 day
- Standard feature with UI + API

**5 points** = 1-2 days
- Feature with some complexity

**8 points** = 2-3 days
- Complex feature with multiple components

**13 points** = 3-5 days
- Very complex feature, external integration

**21 points** = 5+ days (should be broken down)
- Epic-level work, needs decomposition

---

## Total Estimate

**Total Story Points:** 240 points
**Estimated Days:** 60 developer-days (240 points / 4 points per day)
**With 3 Developers:** 20 calendar weeks (~5 months)
**With Buffer (20%):** 24 weeks (~6 months) ✅

---

## Definition of Done (DoD)

A user story is considered "Done" when:

- [ ] Code written and reviewed (PR approved)
- [ ] Unit tests written (>80% coverage for new code)
- [ ] Integration tests written (for API endpoints)
- [ ] UI matches design mockups
- [ ] Responsive on mobile, tablet, desktop
- [ ] Accessibility requirements met (keyboard nav, ARIA labels)
- [ ] Error handling implemented
- [ ] Loading states implemented
- [ ] Documentation updated (API docs, README)
- [ ] Deployed to staging environment
- [ ] QA tested and approved
- [ ] Product owner accepts

---

## Risk Mitigation

### High-Risk Stories

**Plaid Integration (Story 3.1)** - 21 points
- **Risk:** API complexity, bank connectivity issues
- **Mitigation:** Start early, use Plaid sandbox extensively, build fallback for manual entry

**Google Calendar Sync (Story 2.5)** - 13 points
- **Risk:** OAuth complexity, sync conflicts
- **Mitigation:** Thorough testing with multiple calendars, handle edge cases

**Recurring Events (Story 2.4)** - 13 points
- **Risk:** Complex logic, edge cases (leap years, DST)
- **Mitigation:** Use battle-tested library (rrule.js), extensive unit tests

### Dependencies

- **SendGrid Account:** Needed by Week 1 (email verification)
- **Plaid Account:** Needed by Week 6 (setup takes time)
- **Google OAuth:** Needed by Week 5 (approval process)
- **AWS S3 / Storage:** Needed by Week 16 (profile photos)

---

## Success Metrics (Post-MVP Launch)

**After 3 Months:**
- 100+ active families
- 70% weekly active user rate
- Average 3+ family members per account
- 80% of bills marked as paid on time
- 50% of users have linked at least 1 bank account

**After 6 Months:**
- 500+ active families
- Net Promoter Score (NPS) > 40
- < 10 critical bugs in production
- Average session duration > 5 minutes
- Feature adoption > 60% for core features

---

## Next Steps

1. **Product Owner:** Prioritize and refine stories
2. **Tech Lead:** Review estimates and technical approach
3. **Team:** Break down 13+ point stories if needed
4. **Scrum Master:** Set up project tracking (Jira/Linear)
5. **Start Sprint 1:** Week 1 - Epic 1 begins!

---

**This MVP delivers value to families while remaining achievable in 6 months with a small team.**
