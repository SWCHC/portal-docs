# Admin Dashboard — Finances

Route: `/dashboard/billing`  
Navigation label: "Finances"

---

## Overview

The Finances page is a reporting tool that generates income and payroll summaries for any selected date range. It has two modes: Customer (income) and Staff (payroll). Both modes filter by `cleaning_date` — the same date field used by the cleaning calendar — so the numbers always match what Admins see on the calendar.

Date ranges and mode selection persist in the URL as query parameters (`?start=...&end=...&mode=...`), so the report is restored automatically on page refresh.

---

## Date Range Selection

- Start date and end date pickers (no labels, just placeholder text)
- "Show Totals" button generates the report
- Recent expenses (last 7 days) are shown by default before a report is generated

---

## Invoicing Cadence

Customers are invoiced every Tuesday for the previous 7 days. One invoice per customer covers:
1. Turnover fees for all cleanings in the period
2. Any `booking_additional_billing` entries tied to those cleanings
3. Any additional hours charges (`booking_additional_hours`) for those cleanings

Invoicing itself is handled outside the portal (the portal tracks the numbers; actual invoices are sent separately).

---

## Customer Mode

Shows income broken down by customer, then by property, then by individual cleaning.

### Summary Card

A summary card appears above the detailed breakdown. It contains:
- **Grand Total Billed** — one prominent number covering the full date range across all customers
- A sortable table listing each property with:
  - Number of cleanings in the period
  - Total dollars billed for that property (turnover fees + additional billing)

Clicking a property row in the summary scrolls to that property's detail card below. A "Back to summary" link appears in the top-right of each property detail card.

Sort columns: Cleanings (count), Total (amount) — click to toggle asc/desc.

### Detailed Breakdown

Below the summary, each customer has a section containing each of their properties, and each property lists its individual cleanings.

For each cleaning row:
- Cleaning date (not check-out date)
- Turnover rate

Below each cleaning row, additional items are shown indented with a ↳ prefix:
- Additional billing entries: amount + note
- Referral fee (if enabled for this property)

**Property subtotal** = sum of turnover rates + sum of additional billing amounts + referral fees for the period

**Customer total** = sum of all property subtotals

### Expense Line Items

Property-level expenses can be added from the Finances page (Customer mode). An expense is tied to a specific property and has: name/description, date, and dollar amount. Expenses appear as line items in the customer's billing detail and are included in totals.

---

## Staff Mode

Shows payroll broken down by staff member, then by individual cleaning or task.

### Summary Card

A summary card appears above the detailed breakdown containing:
- **Total Payroll** — grand total labor cost for the selected date range
- A sortable table listing each staff member with:
  - Number of cleanings
  - Total pay

Clicking a staff member row scrolls to their detail card. A "Back to summary" link appears on each staff detail card.

Sort columns: Cleanings (count), Total (amount).

### Detailed Breakdown

For each staff member, individual cleanings are listed with:
- Cleaning date
- Property address
- Base pay: `hourly_rate × turnover_time`

Below each cleaning, additional items are shown indented with a ↳ prefix and edit/delete icons:
- Additional hours entries: hours × hourly_rate + note
- Tasks are listed separately with: `hourly_rate × task_hours`

**Staff member total** = cleaning pay + task pay + additional hours pay

### Editing Additional Hours from Finances

Each additional hours entry in the staff finance report has a pencil (edit) and trash (delete) icon. Editing switches the row to inline inputs (hours and note) with Save/Cancel. Deleting shows a confirmation dialog before removal.

---

## Pay Calculations (Full Detail)

### Base Cleaning Pay

```
cleaning_pay = hourly_rate × turnover_time
```

`turnover_time` is stored in decimal hours (e.g., 1.5 = 1 hour 30 minutes, minimum 0.25 = 15 minutes).  
`hourly_rate` is the staff member's rate stored on their staff record.

Example: Maria earns $22.88/hr and a property has a 5-hour turnover time.  
`cleaning_pay = $22.88 × 5 = $114.40`

### Task Pay

```
task_pay = hourly_rate × task_hours
```

`task_hours` is entered by admin when creating or editing a task with an assigned staff member. A warning icon appears on the task bar if a staff member is assigned but hours are not yet entered.

### Additional Hours Pay

```
additional_pay = hourly_rate × additional_hours.hours
```

Additional hours are logged per booking, per staff member, from the cleaning detail lightbox. The note field is required. Only verified staff assigned to the cleaning can be selected.

### Referral Fee

```
referral_fee = floor(turnover_rate × 0.10)
```

Opt-in per property. Floored to the nearest dollar (not rounded — always drops the cents).  
Example: turnover_rate = $285.00 → `floor($28.50) = $28.00`

### Additional Billing (Customer Invoice)

A flat dollar amount added to a specific booking's invoice. Entered from the cleaning detail lightbox. Note is required. No payroll impact.

```
customer_total_for_booking = turnover_rate + sum(additional_billing.amount) + referral_fee
```

### Date Filtering

All queries in the Finances page filter by `cleaning_date` (not `check_out`). This ensures that a booking with a checkout on April 8 and a cleaning on April 9 appears in the April 9 column — matching the cleaning calendar exactly.

```sql
WHERE cleaning_date >= start_date AND cleaning_date <= end_date
```

### Monthly Calendar Totals

The cleaning calendar header shows monthly totals calculated from all bookings in the current month:

```
Monthly Income = sum(turnover_rate) + sum(booking_additional_billing.amount)  [for all cleanings in month]
Monthly Labor  = sum(hourly_rate × turnover_time) [all assigned staff, all cleanings]
               + sum(hourly_rate × task_hours) [all tasks]
               + sum(hourly_rate × additional_hours.hours) [all additional hours]
Monthly Net    = Monthly Income - Monthly Labor
```

### Per-Date Financial Bar

Each date cell on the cleaning calendar shows a financial bar calculated from cleanings and tasks on that specific date:

```
Date Income = sum(turnover_rate) + sum(booking_additional_billing.amount) [cleanings on this date]
Date Labor  = sum(hourly_rate × turnover_time) [assigned staff for cleanings on this date]
            + sum(hourly_rate × task_hours) [tasks on this date]
            + sum(hourly_rate × additional_hours.hours) [additional hours for cleanings on this date]
Date Net    = Date Income - Date Labor
```

The bar appears on any date with cleanings OR tasks (not only on dates with checkouts).

---

## Important Notes

- The `cleaning_status` field exists on the `bookings` table but is null for all records. It is not used in any billing or payroll calculations. Staff pay is calculated from assignments (`booking_assigned_staff`), not from cleaning status.
- All dollar amounts are displayed with exactly two decimal places throughout (e.g., $60.00, not $60).
- The staff finance report previously had a bug where it filtered by `cleaning_status = 'completed'`, returning no results. This was fixed — staff pay now includes all bookings in the date range where they were assigned, regardless of status.
