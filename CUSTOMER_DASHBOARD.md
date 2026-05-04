# Customer Dashboard

Route: `/my-dashboard`

Property owners (customers) are routed here automatically on login. They cannot access any admin or staff pages. The dashboard gives them a view of their bookings, their properties, and a way to communicate with the admin team.

> **Why property owners have their own portal:** These owners manage Airbnb properties and know their booking calendars better than the admin does. Rather than having them call or text the admin with every new reservation, the portal lets them enter bookings directly. It appears immediately in the cleaning calendar on the cleaning calendar and can start scheduling. It eliminates a full communication step and cuts down on transcription errors. Owners enter their bookings — the admin handles the cleaning logistics.

---

## Booking Calendar

The booking calendar is the first thing customers see at the top of the dashboard.

### How Bookings Are Displayed

Each property is displayed as a horizontal color-coded bar spanning from check-in to check-out. Colors are auto-assigned per property and remain consistent throughout the session. A color legend ("Index" button) is available to identify which color maps to which property.

> **Why auto-assigned colors, not a color picker:** An earlier version let owners choose their own colors per property. This was removed to simplify the experience for owners who have only one or two properties and don't need to think about it. The system assigns distinct colors automatically and the Index button is there if anyone needs to look up which color is which.

**Bar positioning:** Bars start at approximately 55% across the check-in date cell and end at approximately 45% across the check-out date cell. This creates a visual gap between back-to-back bookings at the same property. Same-property bookings share the same horizontal row across all weeks, so a property's bar always appears in the same vertical position regardless of the week.

**Bar content:**
- Property street address (with unit/suite if applicable)
- Guest name (if one was entered for the booking)
- Tiny bars at the start of a week (checkout-only slivers) do not display labels

**Icons on bars (right-justified):**
- Amber ArrowLeftRight: same-day in and out at this property
- Red Clock: late checkout
- Blue Clock: early check-in

**Same-day turnovers:** When one booking's checkout and the next booking's check-in are on the same day at the same property, an overlap icon (ArrowLeftRight) appears centered on the booking bar at the overlap date.

### Calendar Navigation

Month navigation arrows move forward and backward one month at a time. Date pickers for booking creation default to the month currently visible.

### Adding a Booking

The "New Booking" button opens a booking creation dialog. Fields:
- **Property selector** — shown as radio-style selectors if 5 or fewer properties; searchable list if more than 5. Displays street address only (no city).
- **Check-in date** — date picker with blocked dates (middle days of existing bookings for this property are blocked; check-in and check-out days are allowed to overlap)
- **Check-out date** — same blocking logic
- **Early check-in** checkbox + time picker (same line as late checkout on desktop)
- **Late checkout** checkbox + time picker
- **Owner staying** checkbox — flags that the owner will be present during the stay
- **Guest suite** checkbox — appears if the selected property has a linked bookable guest suite; creates a matching booking for the guest suite simultaneously
- **Save** and **Save & Add Another** buttons

No "same-day turnover" checkbox — same-day turnovers are detected automatically based on booking dates.

### Editing a Booking

Clicking an existing booking bar opens an edit dialog with the same fields, plus:
- Property Owner Notes field (visible to admin on the cleaning detail lightbox, labeled "Property Owner Notes")
- Delete option

### Activity Log

All booking creates, edits, and deletes are logged. The log is accessible to admins from the booking calendar view and records who made each change, what changed, and when.

---

## My Properties

Below the booking calendar, each property is shown as a card. Properties are displayed in a single-column layout. Each card shows:

- Street address (with unit/suite)
- City, State, Zip
- **Booked days** — number of days booked in the current month
- **Vacant days** — number of days not booked in the current month
- **Occupancy %** — booked days ÷ total days in the month, as a percentage

These stats are calculated for the currently visible calendar month.

---

## My Tickets

> **Why a ticket system instead of email or text:** When managing multiple property owners, Support questions and issues that come in through texts or emails get lost in threads. The ticket system gives every conversation a thread, a history, and a visible status — so nothing falls through the cracks and both admins and the owner can see the full history of an issue without scrolling through message apps.

Between the booking calendar and the My Properties section, customers have a "My Tickets" section for submitting and viewing support requests.

### Submitting a Ticket

A simple form with:
- Message body (required) — the first 50 characters become the auto-generated subject
- Photo attachments — up to 3 photos per message, with thumbnail previews before sending. Uploaded to the `ticket-attachments` storage bucket.
- No subject field

### Viewing Tickets

Submitted tickets appear as a list. Clicking a ticket opens the full conversation thread, including any admin replies. Photos are displayed inline within message bubbles.

---

## Edit My Account Details

A button at the top of the dashboard labeled "Edit My Account Details" reveals the account editing form. By default, no personal information is displayed — everything is behind this button.

Editable fields:
- Name
- Email
- SMS Phone (formatted (XXX) XXX-XXXX)
- Street Address, City, State/Province, Zip/Postal Code, Country

---

## What Customers Cannot See

- Other customers' bookings or properties
- Staff names, schedules, or contact information
- Admin notes on their bookings
- Guest names entered by admin (if different from their own entry)
- Cleaning assignments, staff verification status, or any operational detail
- Any admin dashboard page
- Finances, payroll, or other customers' data (enforced by RLS at the database level)

---

## Admin Preview

Admins can preview exactly what a customer sees by clicking "View Customer Dashboard" from the customer detail page or from the admin booking calendar. This opens the customer's dashboard at `/customer-view/:id` in the same window, with a sticky admin preview bar at the top showing "Admin Preview" and a "Return to Dashboard" button. The admin preview does not affect any data — it is read-only from the customer's perspective.
