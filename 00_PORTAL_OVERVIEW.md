# Airbnb Turnover Management Portal — Overview

**Last documented:** May 2026  

---

## What This Portal Is

The Airbnb Turnover Management Portal is a custom-built operations platform for short-term rental turnover and property care businesses. It handles scheduling, staff assignment, customer communication, financial reporting, and payroll — all in one place.

The portal is the operational backbone of the business, used daily by the admin team.

Every feature in this portal grew out of a real operational need — something previously handled via text messages, spreadsheets, or phone calls that was worth automating. The "why" behind every major design decision is documented in `BUSINESS_CONTEXT.md`.

---

## Planned User Guides

Three user guides are planned to help onboard new users to the portal. Each is written for a specific role:

- **Admin Guide** — The admin's day-to-day workflow: morning verification routine, assigning cleanings, reviewing staff check-ins, running Tuesday invoices, managing time-off requests.
- **Staff Guide** — How cleaners use the dashboard: finding their schedule, using the cleaning card, checking in/out, submitting notes, requesting time off.
- **Customer Guide** — How property owners use the portal: entering bookings, reading their calendar, using the ticket system, updating their account.

These guides should use plain language and the admin's operational vocabulary (not technical terms).

---

## Technology Stack

| Layer | Tool |
|---|---|
| Frontend framework | React + TypeScript (built via Lovable) |
| Backend / database | Supabase (PostgreSQL) |
| Authentication | Supabase Auth |
| Serverless functions | Supabase Edge Functions |
| File storage | Supabase Storage |
| Data fetching | React Query (30-second cache) |
| UI components | shadcn/ui + Tailwind CSS |
| Routing | React Router |
| Hosting | Lovable Cloud |

---

## User Roles

There are four roles in the system. Login automatically routes each role to their correct dashboard.

| Role | Access | Dashboard Route | Notes |
|---|---|---|---|
| Admin | Full access to everything | `/dashboard` | Designated admin accounts. Auto-assigned on signup via database trigger. |
| Auditor | Read-only access to the admin dashboard | `/dashboard` | All buttons, edits, and deletes are disabled. A "Read Only" badge appears in the navigation. Currently used in certain read-only contexts. May be assigned to staff later. |
| Staff | Access to staff dashboard only | `/staff-dashboard` | Cleaners and curbside staff. Cannot access any admin pages. |
| Customer | Access to customer dashboard only | `/my-dashboard` | Property owners. Cannot access any admin pages. |

Role is stored in the `user_roles` table. The `has_role()` function is used throughout the app for permission checks. Row-Level Security (RLS) policies enforce role-based data access at the database level.

---

## The Three Dashboards

The portal has three distinct user-facing experiences:

**Admin Dashboard** (`/dashboard` and sub-routes)  
The admin's full operations view. Contains the cleaning calendar, booking calendar, customer management, property management, staff management, finances, tickets, days off, and settings. This is where all scheduling, assignment, and financial tracking happens.

**Staff Dashboard** (`/staff-dashboard`)  
What cleaners see when they log in. Shows their assigned cleanings and tasks on a calendar, with cleaning cards that include property details, door codes, check-in/check-out buttons, and note fields. Staff also submit time-off requests here.

**Customer Dashboard** (`/my-dashboard`)  
What property owners see when they log in. Shows a color-coded booking calendar for their properties, property stats (occupancy, vacant days), and a support ticket interface.

---

## URL Map

| Route | Who Sees It | What It Is |
|---|---|---|
| `/` | Everyone (pre-login) | Login page ("Management Portal") |
| `/reset-password` | Everyone | Password recovery |
| `/dashboard` | Admin, Auditor | Main admin dashboard — cleaning calendar |
| `/dashboard/customers` | Admin, Auditor | Customer list |
| `/dashboard/customers/:id` | Admin, Auditor | Customer detail page |
| `/dashboard/properties` | Admin, Auditor | Properties list |
| `/dashboard/properties/:id` | Admin, Auditor | Property detail page |
| `/dashboard/staff` | Admin, Auditor | Staff list |
| `/dashboard/staff/:id` | Admin, Auditor | Staff detail page |
| `/dashboard/tickets` | Admin, Auditor | Ticket inbox |
| `/dashboard/billing` | Admin, Auditor | Finances / payroll reports |
| `/dashboard/days-off` | Admin, Auditor | Time-off request management |
| `/dashboard/settings` | Admin only | Admin accounts and password management |
| `/my-dashboard` | Customers | Customer booking dashboard |
| `/staff-dashboard` | Staff | Staff cleaning calendar |
| `/customer-view/:id` | Admin only | Admin preview of a customer's dashboard |
| `/staff-view/:id` | Admin only | Admin preview of a staff member's dashboard |

---

## Database Tables

### Core Data

**`profiles`** — Auto-created on signup via Supabase trigger. Links auth user to app data.

**`user_roles`** — Stores role assignments. One row per user per role. Uses the `app_role` enum: `admin`, `auditor`, `staff`, `customer`. The `has_role()` security definer function checks this table.

**`customers`** — Property owners. Fields include name, email, SMS phone, street address, city, state, zip, country, notes, active (portal access toggle), user_id (links to auth).

**`staff`** — Cleaners and admins. Fields include first/last name, email, SMS phone, backup phone, street address, city, zip, Google Maps URL, hourly rate, hire date, role, active (portal access toggle), user_id.

**`properties`** — Individual rental properties. Key fields:
- `address`, `unit_suite`, `city`, `state`, `zip`
- `short_name` — abbreviated label shown on calendar bars
- `turnover_rate` — what the customer is billed per cleaning
- `turnover_time` — estimated hours for the cleaning (in decimal hours, e.g., 1.5 = 1h 30min; minimum increment 0.25)
- `default_check_in_time` / `default_check_out_time` — property's standard times (stored as HH:MM, displayed as AM/PM)
- `referral_amount` — auto-calculated at 10% of turnover_rate, floored to the nearest dollar. Opt-in per property.
- `curb_service` — boolean, indicates if curb service is included
- `laundry` — boolean, indicates if laundry is included
- `bar_color` — color used for booking bars on the customer dashboard calendar
- `is_guest_suite` — boolean, marks this property as a guest suite linked to a primary suite
- `primary_suite_id` — UUID reference to the primary suite if this is a guest suite
- `guest_suite_bookable_together` — boolean, allows booking both units simultaneously
- `combined_cleaning_rate` — rate when both suites are cleaned together
- `archived` — soft-delete; archived properties are hidden from main lists but preserved in history
- `airbnb_link`, `vrbo_link`, `custom_links`

### Booking and Scheduling

**`bookings`** — Central scheduling table. One row per guest stay. Key fields:
- `property_id`, `customer_id`
- `check_in`, `check_out` — original guest dates (never changed after creation)
- `cleaning_date` — the actual scheduled cleaning date. May differ from `check_out` if the admin moves it. Must be between `check_out` and the next guest's `check_in` (exclusive). Defaults to `check_out`. All calendar and billing logic uses `cleaning_date`, falling back to `check_out` if null.
- `original_check_in`, `original_check_out` — preserved when dates are edited, for comparison
- `guest_name` — optional, hidden from staff
- `late_checkout` — boolean
- `late_checkout_time` — HH:MM format
- `early_checkin` — boolean
- `early_checkin_time` — HH:MM format
- `owner_staying` — boolean, owner is present during the stay
- `same_day_turnover` — boolean (legacy field; same-day turnovers are now detected dynamically)
- `notes` — property owner-entered notes (label: "Property Owner Notes")
- `notes_by` — `'owner'` when customer entered the notes
- `admin_notes` — admin-entered notes (not visible to property owners)
- `admin_notes_read` — boolean, staff must acknowledge admin notes before checking in; resets to false when admin updates notes
- `staff_notes` — notes entered by the assigned cleaner
- `staff_notes_attachments` — JSONB array of photo URLs
- `cleaning_status` — status field (note: as of April 2026, this field is null for all bookings; not used in billing calculations)
- `cleaner_check_in`, `cleaner_check_out` — timestamps when staff checked in/out via the staff dashboard
- `needs_review` — boolean flag, set when booking details change after initial entry; shows a warning badge on admin calendar cards

**`booking_assigned_staff`** — Multi-cleaner assignment table. One row per cleaner per booking. Fields: `booking_id`, `staff_id`, `verified` (boolean), `moved_from_booking_id` (UUID — set when a cleaner is reassigned from another cleaning on the same day; triggers purple bar color until verified).

**`checkout_verifications`** — Tracks who has been verified for each booking checkout. One row per person per booking. Key fields: `booking_id`, `checkout_date`, `person_key` (e.g., `checkout-customer`, `staff-{uuid}`), `verified_at`, `verified_by`. Unique constraint on `(booking_id, person_key)`.

**`calendar_events`** — Ad-hoc tasks assigned to a date and optionally a property and staff member. Fields: `title`, `description`, `start_date`, `assigned_staff_id`, `property_id`, `task_hours` (decimal hours for payroll). Tasks show on both the admin cleaning calendar and the assigned staff member's dashboard.

### Financial

**`booking_additional_hours`** — Extra hours logged against a booking for a specific staff member. Fields: `booking_id`, `staff_id`, `hours`, `note` (required). Only verified staff can be selected. Included in payroll calculations.

**`booking_additional_billing`** — Extra charges billed to the customer for a specific booking. Fields: `booking_id`, `amount` (dollar), `note` (required). No payroll impact. Included in customer invoice totals.

### Property Management

**`property_door_codes`** — Door access codes per property. Fields: `property_id`, `description` (title-cased), `code`. Multiple codes per property supported. Codes are masked in the UI (hold-to-reveal).

**`property_change_log`** — Audit trail for property edits. Fields: `property_id`, `changed_by_name`, `changed_by_email`, `changed_by_user_id`, `description`, `created_at`.

### Communication and Support

**`support_tickets`** — Customer support requests. Subject auto-generated from first 50 characters of the message.

**`ticket_messages`** — Individual messages in a ticket thread. Fields: `ticket_id`, `sender_id`, `body`, `attachments` (JSONB array of photo URLs from the `ticket-attachments` storage bucket).

**`booking_activity_log`** — Logs all booking creates, updates, and deletes. Fields: `booking_id`, `changed_by_email`, `changed_by_name`, `description`, `created_at`.

**`master_activity_log`** — Unified audit log covering bookings, property changes, tasks, and verification events. Accessible from a scroll icon in the admin navigation header.

### HR

**`time_off_requests`** — Staff time-off requests. Fields: `staff_id`, `start_date`, `end_date`, `reason`, `status` (`pending` / `approved` / `denied`), `created_at`, `updated_at`.

### Storage Buckets

| Bucket | Purpose |
|---|---|
| `property-photos` | Property images uploaded from the property detail page. Admin-only write access. |
| `ticket-attachments` | Photos attached to support ticket messages. Customers and admins can upload. |

---

## Key Business Rules (Global)

These apply across the entire portal and are documented in detail in the relevant section files.

- All calendar and billing logic uses `cleaning_date` as the authoritative date for a booking, falling back to `check_out` if `cleaning_date` is null.
- A cleaning can be rescheduled to any date from `check_out` up to (but not including) the next guest's `check_in` at that property.
- Staff cannot be assigned to a cleaning if they have an approved day off on that date. Approval of a day off retroactively unverifies any existing assignments for that staff member on the affected dates.
- Turnover time is stored in decimal hours. Minimum practical increment is 0.25 (15 minutes).
- Referral fees are calculated as 10% of `turnover_rate`, floored to the nearest dollar (e.g., $28.70 becomes $28.00). Opt-in per property.
- Customers are invoiced every Tuesday for the previous 7 days, including turnover fees, additional billing, and any additional hours charges.
- Guest names are never shown to staff. Staff see the property owner's name instead.
- Admin notes require explicit staff acknowledgment before the staff member can mark a cleaning as checked in.
- Dollar amounts are displayed with two decimal places throughout (e.g., $60.00, not $60).
- Phone numbers are formatted as (XXX) XXX-XXXX throughout the portal.
- Door code descriptions are stored and displayed in Title Case.
                                                                                                                                                                                                                                                                                            