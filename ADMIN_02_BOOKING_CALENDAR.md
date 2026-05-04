# Admin Dashboard — Booking Calendar & Verify System

The booking calendar is the second view on the admin dashboard, toggled from the cleaning calendar using the green/blue slider in the toolbar. It shows all guest check-ins and check-outs across every property for the current month.

Route: `/dashboard` (toggle to Booking mode)

---

## Purpose

The cleaning calendar tracks cleanings. The booking calendar tracks the guest stays themselves — when guests arrive and depart. These are related but distinct: a checkout on April 8 might have its cleaning moved to April 9. The booking calendar always shows the original check-out date, not the cleaning date.

The booking calendar is also where customer-specific booking management happens — viewing, adding, and editing individual bookings for a selected property owner.

---

## Default State (No Customer Selected)

When no customer is selected, the booking calendar shows all properties across all customers. Each date cell lists:

- Properties with a **checkout** on that date (shown in blue bars — darker shade)
- Properties with a **checkin** on that date (shown in gray bars — lighter shade)
- If a property has both a checkout AND a checkin on the same day, the same-day in/out icon (amber ArrowLeftRight) appears

Sort order within each date cell: turnovers (in/out on same day) first, then remaining checkouts, then check-ins.

Above the calendar grid, the 12 most recently updated customer records are shown as quick-access cards for fast navigation.

---

## Customer-Specific Booking View

When a customer is selected (via the AJAX search bar or by clicking one of the 12 recent customer cards), the calendar switches to show that customer's bookings only.

In this mode:
- Each property's bookings are shown as color-coded horizontal bars using half-cell positioning
- Bars start at approximately 55% across the check-in date cell and end at approximately 45% across the check-out date cell, creating a visual gap between consecutive bookings
- Each property maintains a consistent vertical row position across all weeks (alphabetical by address), so the same property is always on the same row
- Properties are color-coded; colors are auto-assigned per property
- The property's street address (with unit/suite if applicable) appears on the bar
- Guest name appears on the bar when set
- Tiny bars (checkout-only slivers at the start of a week) do not display text labels

Booking bar icons (right-justified inside each bar, same as cleaning calendar):
- Amber ArrowLeftRight: same-day in and out at this property
- Red Clock: late checkout
- Blue Clock: early check-in

**"View Customer Dashboard"** button appears next to the search bar when a customer is selected. Opens the admin preview of that customer's dashboard.

**"New Booking"** button opens the booking creation dialog (see below).

---

## Adding a Booking (Admin)

The new booking dialog includes:

- **Property selector** — 2-column grid showing all properties for the selected customer. If the customer has 5 or fewer properties, radio-style selectors are shown. If more than 5, a searchable list.
- **Check-in date** — date picker, defaults to the month currently visible in the calendar. Blocked dates: days already occupied by an existing booking for this property (middle days only — check-in and check-out days are allowed to overlap).
- **Check-out date** — same date picker behavior
- **Early check-in** checkbox + time picker (on same line as late checkout, desktop layout)
- **Late checkout** checkbox + time picker
- **Owner staying** checkbox — flags that the property owner will be present during the stay
- **Guest suite** checkbox — appears if the selected property has a linked guest suite. When checked, a second booking is automatically created for the guest suite simultaneously.
- **Save** button
- **Save & Add Another** button — saves the booking, then resets the form while keeping the selected customer, ready for the next booking entry

No guest name field in the admin new booking form. No admin notes or owner notes on the new booking form — notes can be added after creation via the cleaning detail lightbox.

---

## Editing a Booking

Clicking an existing booking bar opens an edit dialog with the same fields as the new booking form, plus:
- Owner Notes field (customer-entered notes)
- Admin Notes field (internal, not visible to customer)
- Delete option

---

## Booking Activity Log

When a customer is selected, an activity log panel appears below the calendar showing all changes made to that customer's bookings. Each entry shows who made the change, what changed, and when. Entries are clickable and navigate to the customer's booking view with that customer pre-selected.

Unread activity log entries show a notification badge on the panel header.

---

## Verify System (Unified Dialog)

Both the cleaning calendar and the booking calendar use a single unified verify dialog. It has two tabs — Cleanings and Bookings — toggled by the same green/blue slider used for the main calendar toggle.

The dialog opens to the appropriate tab depending on which button was clicked:
- "Verify next day" button on the cleaning calendar → opens to Cleanings tab
- "Verify Bookings" button on the booking calendar → opens to Bookings tab

The dialog has a fixed height and scrolls internally.

---

### Verify Bookings Tab

Appears on each date cell of the booking calendar. Purpose: confirm that guests have actually checked out and the property is clear.

**Button states:**
- Red: one or more checkouts for this date are unverified
- Blue with checkmark: all checkouts for this date are verified

**Dialog contents (for a selected date):**

Each property checking out on that date gets its own card. The card is split into two equal columns:

Left column:
- Checkout date and time (using `default_check_out_time` or `late_checkout_time`)
- Cleaning date (if different from checkout, shown in bold with "X day(s) after checkout")
- Next guest check-in date and time ("No upcoming check-in" if none exists)

Right column:
- Customer name with "(Customer)" label
- Customer SMS phone number
- Verify checkbox for the customer

Each checkbox is independent. Checking it immediately saves a row to `checkout_verifications` with `person_key = 'checkout-customer'`. Unchecking immediately deletes that row.

---

### Verify Cleanings Tab

Appears on each date cell of the cleaning calendar via the "Verify next day" button. Purpose: confirm that all staff are confirmed and the property is ready to clean.

Note: "Verify next day" means the button on date X verifies cleanings scheduled on date X+1 (the next day). This is intentional — the admin verifies tomorrow's cleanings the day before.

**Button states:**
- Red: one or more cleanings for the next day have unverified staff
- Blue with checkmark: all staff for all next-day cleanings are verified

**Dialog contents (for the next day's cleanings):**

Each cleaning gets its own card, split into two columns:

Left column:
- Checkout date and time
- Cleaning date (bolded with "X day(s) after checkout" if moved)
- Next guest check-in date and time

Right column:
- Each assigned cleaner gets their own row with name, "(Staff)" label, SMS phone, and a verify checkbox

Each checkbox is independent and maps to `person_key = 'staff-{uuid}'` in `checkout_verifications`. Saves and deletes immediately on toggle.

---

### Verification Persistence

All verifications are stored in the `checkout_verifications` database table with a composite unique constraint on `(booking_id, person_key)`. This means:

- Each person (customer or individual cleaner) has their own independent verification row per booking
- Verifications survive page reloads, sign-outs, and sign-ins
- If a new booking or cleaning is added for a date after verification, the new entry starts unverified and the button reverts to red
- If a staff member's day off is approved for a date where they were verified, their verification is automatically removed

The frontend only updates the UI checkbox state after a confirmed successful database write. If the write fails, an error toast is shown and the checkbox reverts.

---

## Blocked Dates on Date Pickers

When creating or editing a booking, the date pickers block already-occupied dates for the selected property. The rule: only the middle days of an existing booking are blocked. The check-in and check-out days themselves are allowed to overlap (enabling back-to-back bookings). This prevents double-booking the interior days of a stay.
