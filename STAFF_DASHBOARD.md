# Staff Dashboard

Route: `/staff-dashboard`  
Also referred to as: "My Calendar"

Staff members (cleaners and curbside staff) are routed here automatically on login. They cannot access any admin pages. The dashboard is built around the staff member's own assigned cleanings and tasks.

> **Design philosophy:** The staff dashboard is built for cleaners using a phone, often while sitting in a car outside a property. Every piece of information a cleaner needs for a job — the address, how to get there, the door code, the checkout time, the next guest's arrival time — is on a single card. The goal was to eliminate the need to contact the admin for routine job information.

---

## Calendar View

The main view is a monthly calendar showing the logged-in staff member's assigned cleanings and tasks.

### Cleaning Bars

Each cleaning assigned to this staff member appears as a bar on its checkout/cleaning date. Bars show:
- Property street address only (no city)
- In/out icon (amber ArrowLeftRight) if it's a same-day turnover at this property
- Late checkout icon (red Clock) with the actual late checkout time on hover
- Can-go-early icon (green Timer) if the cleaning is on a day with no checkout from this property

### Task Bars

Tasks assigned to this staff member appear as blue bars with a left border. Task bars show:
- Task title
- Assigned staff name (which is always their own name in this view)

### Approved Days Off

Approved time-off periods appear on the calendar as green "Day Off" cells, so the staff member can see their upcoming approved time off alongside their cleanings.

---

## Selecting a Date

Clicking a date on the calendar shows the cleanings and tasks for that date in a detail panel below. Day-by-day navigation arrows move forward or backward through dates.

---

## Cleaning Cards

Each cleaning for the selected date appears as a card. The card contains:

### Property Information
- Property street address (with unit/suite if applicable)
- **Google Maps "Directions" button** — top-right of the card, links to the property's Google Maps URL for navigation
- Property photo — shown as a thumbnail. Tapping opens a lightbox view.
- Default check-out time for the property (e.g., "Checkout: 11:00 AM")
- Default check-in time for the next guest (e.g., "Next check-in: 4:00 PM") — shows the next booking's check-in at this property, not the current booking's check-in
- Late checkout time shown separately and highlighted in amber if applicable

### Door Codes

> **Why hold-to-reveal:** Cleaners view their schedule on a phone in public spaces — parking lots, driveways, building lobbies. A door code that stays on screen is a security exposure. The hold-to-reveal pattern means the code is only visible for the exact moment the cleaner is actively reading it and can't be seen by someone glancing at the screen from a few feet away.

All door codes for this property are listed on the cleaning card with their description and masked code. The staff member must **hold down** the "Show Code" button to reveal the code. It masks again on release (mouse up / touch end). Descriptions are displayed in Title Case.

### Admin Notes

> **Why acknowledgment is required:** If there is something critical to communicate before a cleaner enters a property — water damage in the bathroom, a special access requirement, the owner will be present — the system needs to confirm it was actually read, not just delivered. The acknowledgment button creates that confirmation. If the admin later edits the note, the acknowledgment resets so the cleaner has to confirm the updated version.

If the booking has admin notes (`admin_notes` field), they are displayed prominently on the card. Staff must click **"I have read this note"** before the Check In button becomes available. This acknowledgment is stored as `admin_notes_read = true` on the booking. If an admin later updates the admin notes, `admin_notes_read` resets to false and the staff member must acknowledge again.

### Check In / Check Out

Two buttons control the cleaning status:

**Check In** — Available after admin notes are acknowledged (if any). Clicking records a timestamp in `cleaner_check_in` on the booking and updates the cleaning status to "on site." The admin cleaning calendar reflects this in real time.

**Check Out** — Available after checking in. Clicking records a timestamp in `cleaner_check_out` and marks the cleaning as "completed." The admin calendar reflects this.

**Cancel Check-In** — If a staff member checked in by mistake, they can cancel it. The cancellation is logged to the master activity log.

### Staff Notes

> **Why staff notes exist:** Cleaners are the first people to see conditions inside a property after a guest checks out. If there's damage, missing items, or something that needs replacing, that information needs to get back to admin — but it shouldn't automatically go to the property owner before Admins have a chance to review it. Staff notes are a direct internal channel between the cleaning team and the admin.

A "Staff Notes" text area appears on each cleaning card. The staff member can type notes about the cleaning (e.g., damage, items that need to be replaced, things to note for admin). Notes are saved to `bookings.staff_notes` and are visible to admins on the cleaning detail lightbox.

**Photo attachments** — Staff can attach photos to their notes. Uploaded photos are stored in `bookings.staff_notes_attachments` (JSONB array of URLs). Admins can see these photos from the cleaning detail lightbox.

**Editing notes** — Staff can edit their notes after saving. Notes are not visible to property owners.

### Owner Name

> **Why owner name, not guest name:** Cleaners clean properties — they don't need to know who the guest is. In many short-term rental arrangements, there are privacy reasons for that information not to flow beyond the direct guest-host relationship. The property owner's name is useful context (it tells the cleaner whose property they're at). The guest name isn't operationally necessary for the job.

The property owner's name is shown on the cleaning card. Guest names are never shown to staff.

---

## Task Cards

Tasks assigned to this staff member for the selected date appear below cleaning cards. Task cards show:
- Task title
- Task description/notes
- Property (if assigned to a specific property)
- Date

Tasks assigned to staff also appear in the staff's calendar cells (sky-blue bars), the same as on the admin cleaning calendar.

---

## Time Off Requests

A **"Request Time Off"** form is accessible from the staff dashboard. Fields:
- Start date (required)
- End date (required)
- Reason (required)

The form has a close X button and no "Start/End Date" labels — just the date pickers. Submitted requests appear in the admin Days Off page as pending.

Staff can see their own requests and their current status (pending / approved / denied) on the dashboard. Approved days off appear as green calendar cells.

---

## Edit My Contact Info

A button labeled "Edit My Contact Info" reveals an editable form for the staff member's own contact information. By default, no contact info is shown — it's hidden behind this button. Staff can update their own phone numbers and address.

---

## What Staff Cannot See

- Guest names (shown as property owner name instead)
- Admin notes content before acknowledging
- Door codes without holding the reveal button
- Other staff members' cleanings or schedules
- Any admin dashboard page
- Finances, payroll, or customer information
- Other customers' or staff members' data (enforced by RLS at the datab