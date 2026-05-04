# Admin Dashboard — Cleaning Calendar

The cleaning calendar is the default view when an admin logs in. It is the primary operational tool admins use every day to manage cleanings, assign staff, track progress, and monitor finances.

> **Why this is the home screen:** The admin's first job every morning is knowing what's happening that day — which properties need cleaners, which ones are covered, and which ones have same-day turnovers that need to be prioritized. The cleaning calendar is built to answer all of those questions at a glance without clicking into anything.

Route: `/dashboard`  
Toggle: Cleaning (default) | Booking — switched via a green/blue slider in the toolbar.

---

## View Modes

The cleaning calendar has two display modes, toggled by icons in the toolbar:

**Month View (default)** — A full calendar grid showing the current month. Each date cell contains bars for all cleanings and tasks scheduled on that date. Cell height expands dynamically to fit all entries.

**Day View** — A single large date tile for the selected date. Shows all cleanings and tasks for that day with full detail. Navigation arrows move forward or backward one day at a time.

The month/day toggle persists until changed.

---

## Month Header

The header above the calendar grid shows:

```
[Month Year]  [X cleanings · Y same day in and outs — Z manually scheduled]
Income: $X,XXX.XX · Labor: $X,XXX.XX · Net: $X,XXX.XX
```

**Cleanings count** — total number of bookings with a cleaning scheduled in the current month.

**Same day in and outs** — count of properties that have both a check-out AND a check-in on the same calendar date within the month. These are same-day turnovers where the cleaner has no gap between one guest leaving and the next arriving.

**Manually scheduled** — count of cleanings where `cleaning_date` was changed from the original `check_out` date (i.e., the admin moved the cleaning to a different day).

**Monthly totals:**
- Income = sum of `turnover_rate` for all cleanings in the month + sum of any `booking_additional_billing` amounts for cleanings in the month
- Labor = sum of (`hourly_rate` x `turnover_time`) for each assigned staff member across all cleanings + sum of (`hourly_rate` x `task_hours`) for all tasks + sum of (`hourly_rate` x `hours`) for all additional hours entries
- Net = Income - Labor

---

## Date Cell Layout

Each date cell in the month view contains, from top to bottom:

1. **Date number** (top-left)
2. **Verify next day button** — appears when there are cleanings the following day (see Verify System section)
3. **Financial bar** — income (green), labor (red), net — shown when there are cleanings or tasks on this date
4. **Cleaning bars** — one per booking, sorted (see Sorting below)
5. **Task bars** — below cleanings, visually distinct with a blue left border

---

## Cleaning Bar Colors

> **Why color-coded:** Admins need to know at a glance which properties are covered, which need attention, and which are fully confirmed — without clicking into each one. Red means a problem that needs fixing before that cleaning date. Yellow means a cleaner is scheduled but Admins haven't verified them yet. Green means the job is covered and confirmed. The color tells the whole story.

Bar color indicates the assignment and verification status of the cleaning:

| Color | Meaning |
|---|---|
| Red | No cleaner assigned |
| Yellow | Cleaner(s) assigned, but not all are verified |
| Green | All assigned cleaners are verified |
| Purple | At least one assigned cleaner was moved from another cleaning on the same day (reassigned). Reverts to green once verified. |

---

## Cleaning Bar Content

Each bar displays:

```
[Property Short Name or Address]
    - Cleaner First Name L.
    - Cleaner First Name L.   (if multiple)
```

The property's `short_name` is used if set; otherwise the street address is shown.  
Multiple assigned cleaners each appear on their own indented line in "FirstName L." format (e.g., "Maria S.").

---

## Icons on Cleaning Bars

> **Why these four icons:** Late checkouts and early check-ins directly affect how much time a cleaner has to do their job. Same-day in/out turnovers are the most time-sensitive cleanings in the operation — the cleaner has a fixed window between one guest leaving and the next arriving. The "can go early" icon is the positive version of urgency: it tells the cleaner they have flexibility and don't need to wait for a departing guest. All four reduce the number of calls needed to brief the cleaning team.

Icons appear inside the bar, right-justified. They always appear in this order when present:

| Icon | Color | Meaning |
|---|---|---|
| ArrowLeftRight (⇄) | Amber circle | Same-day in and out — a guest is checking out AND a new guest is checking in at this property on the same day. The cleaner has no gap. |
| Timer (⏱) | Green circle | Can go early — the cleaning is scheduled on a day when there is no guest checking out of this property. The cleaner does not have to wait for a guest to leave. This appears when `cleaning_date` differs from `check_out` and no checkout exists for this property on the cleaning date. |
| Clock (🕐) | Red circle | Late checkout — the departing guest has a late checkout. The cleaner must wait until the late checkout time before starting. |
| Clock (🕐) | Blue circle | Early check-in — the incoming guest has an early check-in. The cleaner has less time than usual to complete the turnover. |

All icons have tooltips showing the exact time (e.g., "Late checkout: 1:00 PM") on hover.

If a cleaning's `cleaning_date` is before its `check_out` date (moved earlier than checkout day), the label **"Early"** appears on the bar.

---

## Task Bars

Tasks created via the "Add Task" button appear below cleanings in each date cell. They are visually distinguished by a blue left border. Task bars show:

```
Task Title
    - Assigned Staff First Name L.   (if staff is assigned)
```

A warning icon (⚠️) with tooltip "Hours need to be entered for this task" appears next to the task title if a staff member is assigned but no `task_hours` have been entered.

---

## Sorting Order Within a Date Cell

Cleanings and tasks are sorted in this priority order within each date cell:

1. Early entry cleanings (can go early — no checkout on cleaning day)
2. Same-day in and out turnovers
3. Regular cleanings (alphabetical by address)
4. Tasks (below all cleanings)

---

## Icon Legend

A legend appears below the calendar grid explaining all four icons:
- Amber circle + ArrowLeftRight: "Same-day In & Out"
- Green circle + Timer: "Can Go In Early (no checkout)"
- Red circle + Clock: "Late Checkout"
- Blue circle + Clock: "Early Check-in"

---

## Cleaning Detail Lightbox

Clicking a cleaning bar in any view opens a detail lightbox. It contains:

**Booking Info (top section):**
- Property name / short name and owner name
- Duration (turnover time)
- Check-in: date and time (uses `default_check_in_time` from the property)
- Check-out: date and time (uses `late_checkout_time` if late checkout, otherwise `default_check_out_time`)
- Cleaning date — if different from check-out, shown in bold with "(X day(s) after checkout)"
- Late checkout indicator (if applicable)
- Next guest check-in: date and time of the next booking at this property (blue highlighted box, shown below the current booking's dates)

**Assigned Cleaners section:**
- Lists each assigned cleaner with their verification status
- Unverified: amber exclamation icon
- Verified: green checkmark
- Day off conflict: red warning icon with tooltip "This staff has an approved day off on this date"
- Moved from another cleaning: purple indicator
- Actions: verify/unverify toggle, move cleaner to another same-day cleaning

**Door Codes section:**
- All door codes for this property, listed with their description
- Code is visible (not masked in the lightbox)

**Notes section:**
- "Property Owner Notes" — entered by the customer
- "Notes for this cleaning" (Admin Notes) — entered by admin, not visible to customers

**Additional Hours to Pay Staff:**
- Lists any additional hours logged for this cleaning
- Each entry shows: staff name, hours, note, edit/delete buttons
- Add form: staff dropdown (verified staff only), hours input, required note
- Only verified assigned staff can be selected

**Additional Amount to Bill Customer:**
- Lists any additional billing entries for this cleaning
- Each entry shows: amount, note, edit/delete buttons
- Add form: dollar amount, required note
- No payroll impact — customer invoice only

**Cleaning Date Editing:**

> **Why this window exists:** Sometimes Admins need to schedule a cleaning the day after checkout rather than on checkout day itself — maybe the property needs extra time, or the assigned cleaner isn't available until the next morning. The system allows this as long as the cleaning still happens before the next guest arrives. The date picker enforces that window so it's physically impossible to accidentally schedule a cleaning after the next guest is already in the property.

- Admin can change the cleaning date from within the lightbox
- Date picker is restricted: cannot select dates after the next guest's check-in at this property
- Date picker defaults to the month of the current cleaning date
- Changing the cleaning date updates `cleaning_date` only — `check_in` and `check_out` are never modified
- "Original booking: [dates]" shown if dates were previously edited

---

## Adding a Cleaning

The "Add Cleaning" button opens a dialog to create a standalone cleaning (not tied to a booking). Fields:
- Property (AJAX search)
- Date
- Assigned staff (pill-style multi-select)

---

## Adding a Task

The "Add Task" button (appears alongside "Add Cleaning" when a date is selected) opens a dialog. Fields:
- Title (required)
- Description / notes
- Date (required) — saved with time T12:00:00 to prevent timezone off-by-one errors
- Property (AJAX search, optional)
- Assigned staff (optional)
- Hours (appears only when staff is assigned)

---

## Staff Cleanings & Hours Panel

A collapsible panel below the calendar (starts collapsed). Shows each active staff member's stats for the current month:

For each staff member:
- Cleaning count and total cleaning hours and pay (hourly_rate x sum of turnover_time)
- Task count, hours, and pay (hourly_rate x sum of task_hours) — shown as a separate line if the staff member has tasks
- Additional hours count, total hours, and pay — shown separately if present
- A combined total line when multiple categories exist
- Staff with 0 cleanings and 0 hours get a subtle red background

**Pay calculation per staff member (monthly):**
```
Cleaning pay  = sum(hourly_rate x turnover_time) for all assigned cleanings
Task pay      = sum(hourly_rate x task_hours) for all assigned tasks with hours
Additional pay = sum(hourly_rate x additional_hours.hours) for all additional hour entries
Total pay     = Cleaning pay + Task pay + Additional pay
```

---

## Activity Log Panel

A collapsible panel below the Staff Cleanings section (starts collapsed). Shows all booking changes from `booking_activity_log`. Entries include: who made the change, what changed, and when. Unread entries show a notification badge on the panel header. A "Mark all read" button clears the badge.

---

## Verify Next Day System

> **Why this exists:** The admin's standard practice is to contact each customer the day before their checkout to confirm the arrangement, and to confirm with each assigned cleaner. Rather than keeping a separate list or relying on memory, this button is the daily structured checklist for that workflow. Each person who needs to be reached — the customer and each assigned cleaner — gets their own checkbox. The button turns blue when everyone's been confirmed.

Each date cell in the month view has a small button that indicates the verification status of cleanings the following day:

- **Red button ("Verify next day")** — One or more cleanings scheduled for the next day have unverified staff or unverified customer checkouts
- **Blue button ("Verified")** — All cleanings for the next day are fully verified

Clicking the button opens the unified Verify Dialog (documented separately in `ADMIN_02_BOOKING_CALENDAR.md` under the Verify System section).

The button uses `cleaning_date || check_out` to determine which bookings appear — this matches the calendar's own date logic exactly.

---

## Financial Bar (Per Date Cell)

A thin bar appears in each date cell between the verify button and the cleaning bars. It shows:

- Green section: total income for that date (sum of `turnover_rate` + `booking_additional_billing` amounts for all cleanings on this date)
- Red section: total labor for that date (sum of all staff pay + task pay + additional hours pay)
- Net amount shown on the right

The bar appears whenever a date has cleanings OR tasks (not only when checkouts exist).
        