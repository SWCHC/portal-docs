# Admin Dashboard — Days Off

Route: `/dashboard/days-off`  
Navigation: "Days Off" link in the admin nav bar.

---

## Overview

The Days Off section manages staff time-off requests. Staff submit requests from their dashboard; admins approve or deny them here. Approved days off automatically affect the cleaning calendar — any staff assignments on conflicting dates are flagged and unverified.

---

## Admin View

The page has two sections:

### Pending Requests

Lists all time-off requests with a status of `pending`. Each request card shows:
- Staff member name
- Requested date range (start and end date)
- "Requested: [date/time]" timestamp
- Approve and Deny buttons

Approving or denying a request immediately updates the status in the `time_off_requests` table.

### Recent Resolved Requests

Lists recently approved and denied requests. Each card shows:
- Staff member name
- Date range
- Status (approved / denied)
- "Requested: [date/time]"
- "Approved: [date/time]" (if approved)

Note: The reason field for time-off requests is stored in the database but is not displayed in the admin view.

### Calendar View

A calendar showing all approved days off across all staff for the current month, for at-a-glance scheduling awareness.

---

## What Happens When a Request Is Approved

Approving a time-off request triggers two automatic actions:

1. **Assignment conflict check** — The system scans all existing `booking_assigned_staff` records for this staff member on dates that fall within the approved time-off period.

2. **Auto-unverify** — Any verified cleaning assignments found during that scan are set to `verified = false`. Each conflict is logged to the `master_activity_log` with a description of what was unverified and why.

This ensures the cleaning calendar immediately reflects the conflict — previously green bars may turn yellow or red after an approval, flagging the admin to reassign those cleanings.

---

## Assignment Blocking (Forward-Looking)

When an admin tries to assign a staff member to a cleaning on a date when they have an approved day off, the system blocks the assignment and displays: **"This staff is booked off for that day."**

This check applies in every place staff can be assigned:
- Quick-assign on the cleaning calendar
- Cleaning detail lightbox
- Moving a cleaner to another booking
- Adding a standalone cleaning with staff
- Creating or editing a task with assigned staff
- Staff Schedule page assignment

---

## Time-Off Data

Stored in the `time_off_requests` table:

| Field | Description |
|---|---|
| `staff_id` | The staff member making the request |
| `start_date` | First day of requested time off |
| `end_date` | Last day of requested time off |
| `reason` | Optional reason (stored but not displayed in admin view) |
| `status` | `pending`, `approved`, or `denied` |
| `created_at` | When the request was submitted |
| `updated_at` | When the status was last changed (used for "Approved:" timestamp) |
