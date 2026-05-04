# Admin Dashboard — Properties

Route: `/dashboard/properties`  
Detail: `/dashboard/properties/:id`

---

## Properties List

The properties list shows all active properties in a searchable, sortable table. A property count is shown at the top.

### Columns

| Column | Description |
|---|---|
| Address | Street address (with unit/suite if applicable). Default sort. |
| Short Name | Abbreviated label used on cleaning calendar bars |
| City | Property city |
| Customer | Assigned property owner(s) |
| Rate | Turnover rate (billed to customer per cleaning) |
| Referral | Referral fee (shown as calculated amount if referral is enabled for this property) |
| Time | Turnover time in hours (e.g., 1.5 = 1h 30min) |
| Beds | Bedroom count |
| Curb | Curb service included (icon) |
| Laundry | Laundry included (icon) |
| Portal Access | Toggle to enable/disable customer portal access for this property |

Clicking any row navigates to that property's detail page.

### Search and Sort

AJAX search bar with "Clear" button. All column headers are sortable (click to toggle asc/desc).

### Archived Properties

A "View Archived" button toggles visibility of archived properties. Archive/unarchive is available from the property detail page and is logged in the change history. Archiving does not delete bookings or history — it removes the property from active lists.

### Adding a Property

The "Add New Property" button navigates directly to a new property detail page form (not a lightbox). The full detail form is used for new property creation, including photo upload support.

### Edit All Mode

Puts the table into bulk inline editing mode. In Edit All mode:
- Address, Short Name, Rate, Referral (checkbox), Time (bulk editable across all rows), Curb Service (checkbox), Laundry (checkbox), Beds columns are editable
- City and Customer columns are hidden during Edit All
- Default check-out time and default check-in time are bulk-editable
- Changes auto-save on blur (tab or click out)
- "Save All" button at the bottom saves all pending changes
- Column headers repeat every 10 rows for reference
- Table is full-page height (not constrained to a scroll container)

---

## Property Detail Page

Route: `/dashboard/properties/:id`

The detail page is the complete management interface for a single property. It is also used for new property creation.

### Photo

A photo placeholder is shown at the top-left. Admins can upload a property photo via drag-and-drop or file picker. For new properties, the photo is stored locally and uploaded automatically after the property record is saved.

### Customer Assignment

The customer card appears at the top-right (desktop layout). An AJAX search allows assigning or changing the property owner. Multiple customers can be assigned to one property (e.g., co-owners). Each assigned customer can be individually removed. An "Add New Customer" button below the search opens the customer creation lightbox without navigating away.

### Address & Details

- Street Address
- Unit / Suite number
- City, State, Zip
- Short Name — the abbreviated label shown on cleaning calendar bars (e.g., "Rosa Dr" instead of "41818 W Rosa Dr")
- Bedrooms, Bathrooms, Square Footage
- Airbnb Link, VRBO Link, Custom Links

### Turnover & Services

- **Turnover Rate** — amount billed to the customer per cleaning (no input arrows on the field)
- **Turnover Time** — estimated hours for the cleaning (decimal, minimum 0.25 increments). Used in all staff pay calculations.
- **Referral Fee** — checkbox to enable. When enabled, the fee is auto-calculated as `floor(turnover_rate x 0.10)`. Not manually entered. Displayed in the properties list when enabled.
- **Curb Service** — checkbox
- **Laundry** — checkbox
- **Default Check-out Time** — property's standard checkout time (HH:MM, displayed as AM/PM throughout the portal)
- **Default Check-in Time** — property's standard check-in time

### Door Codes

Multiple door codes can be stored per property. Each code has:
- **Description** — auto title-cased (e.g., "Front Door", "Garage Code")
- **Code** — the access code itself

In the cleaning detail lightbox and on the staff dashboard, codes are displayed with their description. On the property detail page, codes are visible in plain text. On the staff dashboard, they are masked with asterisks and must be held to reveal.

Adding, editing, and deleting door codes is logged in the property change history.

### Guest Suite

If this property has a guest suite (a secondary unit on the same property), it can be linked here. This section appears at the bottom of the detail page, above Change History. It is hidden if the property is already designated as a guest suite itself.

Fields:
- **This is a guest suite** — checkbox marking this property as the guest suite of a primary suite
- **Primary Suite** — AJAX search to link this guest suite to its primary property
- **Bookable together** — checkbox indicating the two units can be booked simultaneously
- **Combined Cleaning Rate** — rate when both are cleaned in one visit

When booking a property that has a linked guest suite, an "Include the guest suite in this booking" checkbox appears in both the admin and customer booking dialogs. When checked, a second booking is automatically created for the guest suite with the same dates.

> **Note:** Guest suite details and edge cases to be confirmed — see open question flagged during documentation session.

### Change History

A collapsible section at the very bottom of the property detail page (starts collapsed). Shows a chronological log of all changes made to this property, including:
- Who made the change (name and email)
- What changed (description)
- When it happened

Door code changes, archive/unarchive events, and field edits are all logged here via the `property_change_log` table.

---

## Referral Fee Calculation

The referral fee is per-property opt-in. When enabled:

```
referral_fee = floor(turnover_rate * 0.10)
```

Example: turnover_rate = $285.00 → referral_fee = floor($28.50) = $28.00

The fee is not manually editable. It is always derived from the current turnover rate. It appears as a line item in the customer finance report.

---

## How Properties Connect to the Portal

Properties are linked to customers via a join relationship (multiple customers can share a property). They appear on the customer dashboard under "My Properties." On the admin cleaning calendar, the property's `short_name` is used for calendar bars; if no short name is set, the street address is used.

Properties can also be linked to bookings (which drive the entire scheduling system) and to door codes, photos, expenses, and change history.
