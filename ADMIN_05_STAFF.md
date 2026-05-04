# Admin Dashboard — Staff

Route: `/dashboard/staff`  
Detail: `/dashboard/staff/:id`

---

## Staff List

The staff list shows all active staff members divided into three sections. Counts for each category are shown at the top of the page.

### Sections

| Section | Role Value | Who |
|---|---|---|
| Admin Staff | `admin` | Admin accounts |
| Cleaners | `cleaner` | Cleaning staff |
| Curbside Staff | `curb_service` | Staff who manage garbage/recycling bin service |

Each section has its own table with the same column structure.

### Columns

| Column | Description |
|---|---|
| Name | Staff member's full name |
| SMS Phone | Primary contact number, formatted (XXX) XXX-XXXX |
| Hourly Rate | Pay rate in dollars per hour |
| Hire Date | Date staff member started |
| Last Login | Most recent portal login timestamp |
| Role | Displayed roles (a staff member can have multiple roles) |
| Portal Access | Toggle switch — enables or disables this staff member's ability to log in |

Clicking any row navigates to that staff member's detail page.

### Search and Sort

AJAX search bar with "Clear" button. Column headers are sortable.

### Archived Staff

A "View Archived" button toggles visibility of archived staff. Archived staff do not appear in assignment dropdowns, do not receive task assignments, and are excluded from the Staff Cleanings & Hours panel on the cleaning calendar. Archive/unarchive from the staff detail page.

### Edit All Mode

Bulk inline editing for Hourly Rate and Hire Date columns. Changes auto-save on blur. "Save All" at the bottom of each section table.

---

## Staff Detail Page

Route: `/dashboard/staff/:id`

### Contact Information

- First Name, Last Name
- Email
- SMS Phone (formatted (XXX) XXX-XXXX)
- Backup/Emergency Phone
- Street Address, City, Zip
- Google Maps URL (links to their home address for routing purposes)

### Role

Dropdown selector: Cleaner, Curb Service, Admin. A staff member can hold multiple roles (e.g., a cleaner who is also an admin).

### Employment

- Hourly Rate (dollar amount)
- Hire Date

### Notes

Free-text notes field for admin use. A disclaimer appears below the field: "Notes are not visible to the staff member."

### Admin Notes (Ticketed)

A separate "Admin Notes" section allows admins to add notes that are automatically saved as support tickets in the ticket system. Each note entry includes a message, optional photo attachment, and is logged with the admin's identity. These are accessible from the Tickets section and are distinct from the general Notes field above.

### Portal Access

- **Portal Access toggle** — enables/disables login
- **View Staff Dashboard** button — opens an admin preview of this staff member's dashboard (`/staff-view/:id`), in the same window
- **Send Reset Email** — sends a Supabase Auth password reset email
- **Set Password** — opens a dialog to set a new password directly (via secure edge function requiring admin role)

---

## How Roles Work

Roles are stored in the `user_roles` table (one row per role per user). The `app_role` enum includes: `admin`, `auditor`, `staff`, `customer`.

**Admin auto-assignment:** When designated admin email addresses register via the auth page, a database trigger (`auto_assign_admin`) automatically inserts an `admin` row into `user_roles` and creates a staff record for them.

**Role-based routing on login:**
- Admin → `/dashboard`
- Auditor → `/dashboard` (read-only, "Read Only" badge in nav)
- Staff → `/staff-dashboard`
- Customer → `/my-dashboard`

---

## Staff Assignment to Cleanings

Staff are assigned to cleanings from the cleaning calendar detail lightbox. Key rules:

- Multiple staff can be assigned to one cleaning (stored in `booking_assigned_staff`)
- Each assignment has a `verified` flag (default false)
- If a staff member is moved from one cleaning to another on the same day, `moved_from_booking_id` is set on their new assignment — this triggers a purple bar color on the calendar until verified
- Staff cannot be assigned to a cleaning on a date when they have an approved day off. The system blocks the assignment and shows: "This staff is booked off for that day."
- When an admin approves a day off that overlaps with existing verified assignments, those assignments are automatically unverified and logged to the master activity log

---

## Staff Cleanings & Hours (Cleaning Calendar Panel)

The collapsible Staff Cleanings & Hours panel on the cleaning calendar calculates pay for the current month per staff member:

```
Cleaning pay   = sum(hourly_rate × turnover_time) for all assigned cleanings in month
Task pay       = sum(hourly_rate × task_hours) for all assigned tasks in month
Additional pay = sum(hourly_rate × additional_hours.hours) for all additional hour entries in month
Total pay      = Cleaning pay + Task pay + Additional pay
```

Staff with 0 cleanings and 0 hours for the month are shown with a subtle red background.

---

## Current Staff (as of project start — March 2026)

The portal launched with 16 imported staff members: 15 cleaners and 1 curbside staff (John Pritchard).
