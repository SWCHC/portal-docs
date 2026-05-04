# Admin Dashboard — Settings & Master Activity Log

Route: `/dashboard/settings`  
Navigation: Gear icon in the admin nav bar.

---

## Settings Page

The Settings page is restricted to admin role only (auditors cannot access it). It currently contains one section: administrator account management.

### Administrators

Lists all users with the `admin` role. Each admin account is displayed with action buttons for editing, password reset, and direct password management.

Each admin row has three action buttons:

| Button | What It Does |
|---|---|
| Edit (pencil) | Opens a dialog to update the admin's name and email in the staff table |
| Send Reset Email | Sends a Supabase Auth password reset email to the admin's email address |
| Set Password | Opens a dialog to type and set a new password directly, with a show/hide toggle |

Password operations go through a secure Supabase Edge Function (`admin-set-password`) that verifies the caller has an admin role before using the service role key. Standard users cannot call this function.

### Admin Auto-Assignment

When a designated admin email address registers through the auth page, a database trigger (`auto_assign_admin`) automatically:
1. Inserts an `admin` row into `user_roles` for that user
2. Creates a staff record for them if one doesn't already exist

This means admin accounts do not need to be manually configured at signup — it happens automatically.

---

## Master Activity Log

Accessible via a scroll icon (ScrollText) in the left side of the admin navigation header, to the left of the Sign Out button.

The master activity log is a unified audit trail covering all significant actions taken in the portal. It draws from:
- `booking_activity_log` — booking creates, updates, and deletes
- `property_change_log` — property field edits, archive/unarchive, door code changes
- `master_activity_log` — tasks, cleaning assignments, verification events, day-off conflict unverifications, and other actions

Clicking the icon opens a dialog showing all log entries with:
- Entity type badge (Booking, Property, Task, Verification, etc.)
- Description of what changed
- Who made the change
- Timestamp

The log is read-only. It cannot be edited or deleted from the UI.

### What Gets Logged

| Action | Log Table |
|---|---|
| Booking created / updated / deleted | `booking_activity_log` |
| Property fields edited | `property_change_log` |
| Property archived / unarchived | `property_change_log` |
| Door code added / edited / deleted | `property_change_log` |
| Task created / updated / deleted | `master_activity_log` |
| Cleaning assignment added / removed | `master_activity_log` |
| Cleaner verified / unverified | `master_activity_log` |
| Checkout verified / unverified | `master_activity_log` |
| Day-off approval unverifying an assignment | `master_activity_log` |

---

## Security Notes

- All admin routes (`/dashboard/*`) require admin or auditor role. Non-admin users are redirected to their appropriate dashboard.
- Row-Level Security (RLS) is enforced at the database level — customers and staff can only access their own data, regardless of what the frontend requests.
- The `admin-set-password` edge function requires the caller to have an admin role in `user_roles` before it will use the service role key. This prevents privilege escalation.
- Leaked Password Protection should be enabled in the Supabase Auth dashboard settings (this is a platform setting, not a code setting).
- Storage bucket `property-photos` is admin-write-only. The `ticket-attachments` bucket requires the uploader to own the related ticke