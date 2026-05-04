# Admin Dashboard — Customers

Route: `/dashboard/customers`  
Detail: `/dashboard/customers/:id`

---

## Customer List

The customers list shows all active customers in a searchable, sortable table. An active count is shown at the top (e.g., "X customers").

### Columns

| Column | Description |
|---|---|
| Name | Customer's full name |
| Email | Primary email address |
| SMS Phone | Cell/SMS phone number, formatted (XXX) XXX-XXXX |
| Notes | Short notes preview |
| Portal Access | Toggle switch — enables or disables the customer's ability to log into the customer dashboard |

Clicking any row navigates to that customer's detail page.

### Search

AJAX search bar filters the list in real time. A "Clear" button inside the search field resets it.

### Sorting

Column headers are clickable and toggle ascending/descending sort.

### Archived Customers

A "View Archived" button toggles visibility of archived (inactive) customers. Archived customers are hidden from the main list but preserved in the database. Archive/unarchive is done from the customer detail page.

### Adding a Customer

The "Add Customer" button opens a lightbox with a simplified form: Name, Email, Cell Phone, Notes. SMS phone and address fields are only shown when editing an existing customer.

### Edit All Mode

An "Edit All" button puts the table into bulk editing mode. The Email, Phone, and SMS Phone columns become inline input fields. Changes auto-save when you tab or click out of a field. A "Save All" button at the bottom of the table saves all pending changes at once. "Cancel" reverts to read-only mode. Column headers repeat every 10 rows during Edit All mode for reference.

---

## Customer Detail Page

Route: `/dashboard/customers/:id`

The detail page shows complete information for one customer and is the main editing interface.

### Contact Information

- Name
- Email
- SMS Phone (formatted (XXX) XXX-XXXX) — this is the primary contact number used throughout the portal
- Street Address, City, State/Province, Zip/Postal Code, Country (US/Canada toggle)
- Notes

Saving the detail page navigates back to the customer overview list.

### Linked Properties

A "My Properties" section appears at the top of the detail page, listing all properties assigned to this customer. Each property card is clickable and navigates to that property's detail page. An "Add Property" button links to the new property form.

### Portal Access

Password reset options are available from the detail page: send a reset email (via Supabase Auth) or set a password directly (via a secure admin edge function requiring admin role verification).

### "View Customer Dashboard" Button

Opens an admin preview of this customer's portal view (`/customer-view/:id`), navigating in the same window (not a new tab). This lets the admin see exactly what the customer sees.

---

## How Customers Are Connected to the Portal

Each customer record has a `user_id` field that links to a Supabase Auth account. When a customer is given portal access (active = true) and logs in, they are routed to `/my-dashboard`.

Portal access can be revoked by toggling the switch on the customer list. This does not delete the auth account — it simply flags the customer as inactive. The customer will be redirected away if they attempt to log in while inactive.

---

## Notes on Phone Fields

The portal stores two phone-related fields for customers:
- **SMS Phone** — the primary contact number. Used in verification dialogs, staff communications, and displayed throughout the admin views.
- The older "Phone" field was removed from both the customer detail view and the customer overview. Only SMS phone is shown and used.
