# Admin Dashboard — Tickets

Route: `/dashboard/tickets`  
Navigation: "Tickets" link in the admin nav bar with an unread badge when new messages exist.

---

## What Tickets Are

Tickets are the support communication channel between property owners and the admin team. Customers submit tickets from their dashboard. Admins view and reply from the Tickets page.

Tickets are also used internally for staff admin notes — when an admin adds a note from a staff member's detail page, it is saved as a ticket in this same system.

---

## Ticket List (Admin View)

The left panel shows all open tickets. Each ticket shows:
- Customer name (or staff name for internal notes)
- Auto-generated subject (first 50 characters of the initial message)
- Timestamp of the most recent message
- Unread indicator

Clicking a ticket opens the conversation in the right panel.

---

## Conversation View

The right panel shows the full message thread for the selected ticket. Messages are displayed as chat bubbles.

- Customer messages appear on one side, admin replies on the other
- Photo attachments are displayed inline within the message bubble
- The reply field is at the bottom with a Send button

---

## Photo Attachments

Both customers (from their dashboard) and admins (from the ticket reply interface) can attach photos to messages. Photos are uploaded to the `ticket-attachments` Supabase Storage bucket. The attachment URLs are stored as a JSONB array in the `ticket_messages.attachments` field and rendered inline in the conversation view.

---

## Unread Badge

The "Tickets" nav link shows an unread count badge when there are tickets with unread messages. The admin dashboard also shows an unread tickets section below the activity log panel.

---

## Ticket Subject

There is no subject field in the ticket creation form. The subject is auto-generated from the first 50 characters of the initial message body. This keeps the submission form simple for customers.

---

## Staff Admin Notes (Internal Tickets)

When an admin adds a note from a staff member's detail page (`/dashboard/staff/:id`), the note is saved as a ticket in this system. These appear in the ticket list and can include photo attachments. They are associated with the staff member, not a customer. This allows admins to maintain a documented communication record about each staff member accessible from one place.
