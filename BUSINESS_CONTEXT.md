# Business Context & Design Decisions
## Why the Portal Works the Way It Does

This document captures the reasoning behind the portal's design — the "why" behind features that might otherwise look like arbitrary choices. It's written in plain language and is intended to inform future user guides, onboarding materials, and anyone coming into this project cold.

---

## The Portal in One Paragraph

This portal serves an Airbnb turnover cleaning company. The company runs a team of cleaners who service property owners' short-term rental units. The management portal exists so admins can run that operation without juggling spreadsheets, text threads, and phone calls. Property owners log in to enter their own bookings. Cleaners log in to see their schedule and check in/out. Admins see everything and manage assignments, billing, and communications from a central admin dashboard.

---

## The Closed System Decision

**What:** There is no "Sign up" or "Create an account" option on the login page. Users cannot register themselves.

**Why:** This is a closed, private operation. The company decides who has access. Every property owner and every cleaner gets an account created by Admins, with a password set through the admin panel. The moment the system became real — with actual customer data, door codes, and financial information — self-registration had to go. The login page says "Management Portal" for the same reason: it signals that this isn't a public-facing consumer app.

---

## Why Property Owners Enter Their Own Bookings

**What:** Property owners (customers) log in and add their own check-in/check-out bookings through their dashboard. Admins don't have to enter them manually.

**Why:** These owners each manage one or more Airbnb properties. They know their booking calendars better than the admin does. Rather than having owners call or text the admin with every new reservation, the portal lets them put it directly in the system. It appears immediately in the cleaning calendar so scheduling can begin. It cuts out a full communication step and eliminates transcription errors.

---

## The Three-Dashboard System

**What:** Three completely separate dashboards — one for admins, one for customers (property owners), one for staff (cleaners). Login automatically routes each user to the right one. Nobody can navigate to a dashboard they don't belong in.

**Why:** These three groups need completely different things and should never see each other's data. Property owners shouldn't see what the company pays its cleaners. Cleaners shouldn't see other customers' properties or other cleaners' schedules. And neither should see anything Admins use to run the business internally. The separation isn't just a design preference — it's enforced at the database level (Row-Level Security), so even if someone guessed a URL, they'd get nothing.

---

## Color-Coded Cleaning Calendar (Red / Yellow / Green)

**What:** Every cleaning bar on the admin calendar is color-coded: red = no cleaner assigned, yellow = cleaner assigned but not yet verified, green = all verified.

**Why:** The admin's first question every morning is "which properties aren't covered yet?" That answer should be visible at a glance — not require clicking into each property one by one. Red is a problem. Yellow needs attention. Green means move on. The system was designed so the color alone tells the story.

---

## The Verify Checkout Workflow

**What:** Every date cell on the cleaning calendar has a "Verify" button. Clicking it shows a list of all properties checking out the next day, with the customer's phone number. Each person involved — the customer and each assigned cleaner — has a separate checkbox. The button turns blue once everything is checked off.

**Why:** The admin's standard practice is to contact customers the day before their checkout to confirm the arrangement, and to confirm with each assigned cleaner. This button is the daily checklist for that workflow — a structured way to track who's been reached without relying on memory or a separate notepad. The per-person checkboxes exist because reaching the customer and reaching the cleaner are two separate actions.

---

## Why the Cleaning Date Can Be Moved

**What:** When the admin assigns a cleaning, the default date is the booking's checkout date. But that date can be moved forward — as long as it stays within the window between checkout and the next guest's check-in.

**Why:** Sometimes a property needs to be cleaned the day after checkout, not on checkout day itself. Maybe the property needs extra time. Maybe a cleaner isn't available on checkout day but is available the next morning before the next guest arrives. The portal enforces the window (can't go past the next check-in) so Admins can't accidentally schedule a cleaning after the next guest is already in the property.

---

## Door Code Security (Hold to Reveal)

**What:** Door codes on the staff cleaning card are masked. Cleaners have to physically hold down the "Show Code" button to see the code. The moment they release it, it masks again.

**Why:** Cleaners are viewing their schedule on a phone, often in public spaces — parking lots, driveways, lobbies. A door code that stays visible on screen is a security risk. The hold-to-reveal pattern means the code is only visible for the exact moment a cleaner is actively using it. It can't be read by someone glancing over their shoulder.

---

## Why Guest Names Are Hidden from Cleaners

**What:** Cleaning cards on the staff dashboard never show the guest's name. They show the property owner's name instead.

**Why:** Cleaners clean properties. They don't need to know who the guest is, and in many cases, there are privacy and liability reasons for that information not to flow beyond the guest-host relationship. The property owner's name is useful context (it tells the cleaner who the property belongs to). The guest name isn't operationally necessary for doing the job.

---

## Admin Notes and the Acknowledgment Requirement

**What:** When the admin attaches admin notes to a booking, the cleaner assigned to that job sees those notes prominently on their cleaning card. A button reads "I have read this note" — and the check-in button stays locked until they click it.

**Why:** If there is something critical to communicate before a cleaner enters a property — water damage in the bathroom, a special access requirement, a note about the owner being present — the system needs to confirm it was actually read, not just delivered. The acknowledgment button creates that confirmation. If the admin later updates the note, the acknowledgment resets and the cleaner has to confirm again.

---

## Staff Notes (Separate from Admin Notes)

**What:** Cleaners can write notes on their own cleaning cards. Those notes go to the admin (visible in the admin cleaning lightbox) but are not visible to the property owner.

**Why:** Cleaners are the first people to see conditions inside a property after a checkout. If there's damage, missing items, or something that needs replacing, that information needs to get back to admin. But it shouldn't automatically land in front of the property owner without admins having a chance to review it first. Staff notes are an internal communication channel between the cleaning team and the admin.

---

## The Referral Fee Calculation

**What:** Each property has a referral fee that's automatically calculated as 10% of the turnover rate, floored to the nearest dollar.

**Why:** When a property owner refers a new client to the company, the company gives them a 10% kickback on each cleaning. The referral fee field is auto-populated from the turnover rate so Admins don't have to calculate it manually for every property. It floors (not rounds) because Admins don't pay partial dollars on referrals.

---

## Why Billing Runs on Tuesdays

**What:** The finance section generates invoices by looking at the previous 7 days. The intended cadence is every Tuesday.

**Why:** Invoices run weekly. Tuesday captures the weekend results plus Monday, which is a common checkout day in the short-term rental world. Running it the same day every week makes it a habit and keeps cash flow predictable.

---

## Property Owner Notes in Bookings (Separate from Admin Notes)

**What:** When a property owner creates or edits a booking, they have a "Property Owner Notes" field. These notes are visible to the admin in the admin cleaning lightbox.

**Why:** Owners sometimes have context that affects the cleaning — they'll be at the property during the stay, there's a specific access issue, the guest is leaving at an unusual time. The notes field gives them a structured way to communicate that without calling the office. It shows up clearly for admins when reviewing cleanings, labeled so it's clear it came from the owner.

---

## Why the Edit All Mode Exists

**What:** On the customer, staff, and property list pages, there's an "Edit All" button that turns every row into inline editable fields. You make all your changes and hit Save All once.

**Why:** When the portal first launches for a new company, there can be many customers, properties, and staff records to populate. Clicking into each record one at a time to update a phone number or hourly rate would have taken hours. Edit All was designed for that initial data-entry sprint and remains useful any time contact info or rates need to be updated in bulk.

---

## The Activity Log

**What:** Every booking change, property edit, door code update, and cleaning assignment is logged automatically. The log records who did it and when. Admins can access it from the master activity log icon in the header.

**Why:** With multiple property owners entering their own bookings, admins need to know if something changes. If a booking date shifts or a booking disappears, the log shows exactly when it happened and who did it. It also protects everyone — if there's ever a dispute about what was booked, the log is the record.

---

## Why the Portal Tracks Time Off Against Assignments

**What:** When Admins approve a staff member's time off request, the system automatically unverifies any cleaning assignments that overlap with those dates. Admins are notified by the change in color on the calendar.

**Why:** If a cleaner is verified on a job and then Admins approve their vacation after the fact, that job suddenly has no real coverage. Rather than letting that slip through, the system catches it immediately — the cleaning goes back to yellow (unverified) or red (unassigned), prompting the admin to reassign. The time off request can't be buried in a separate system while the cleaning calendar stays green.

---

## The Guest Suite System

**What:** Some properties have a bookable guest suite attached. When a booking is created for the main property, there's an option to include the guest suite in the same booking. If selected, it creates a separate linked booking for the suite automatically.

**Why:** Some properties have a main house and an attached suite that can be rented together or separately. When they're rented together, both need to be cleaned. Linking them at the booking level ensures the cleaning calendar reflects both jobs, and billing captures both turnover rates.

---

## Owner Staying Checkbox

**What:** When adding a booking, there's a checkbox for "Owner staying" that flags that the property owner will be present during the stay.

**Why:** Owner-occupied stays change how a cleaner approaches the job. It's relevant information for scheduling and for what the cleaner should expect when they arrive. The flag surfaces in the cleaning detail so admins and the cleaning team know before anyone shows up.

---

## Portal Access Toggle

**What:** Each customer and staff member has a "Portal Access" toggle. Turning it off blocks their login without deleting their data.

**Why:** Customers come and go, staff turn over. When someone leaves the roster, Admins need to revoke their access without erasing their history — their past bookings, their cleaning records, their billing. The toggle cuts the access cleanly while preserving everything else.

---

## The Auditor Role

**What:** There's a fourth role called "auditor" that gets full read access to the admin dashboard but can't create, edit, or delete anything.

**Why:** Admins may want to give a bookkeeper, a business partner, or a future employee visibility into the portal without giving them the ability to change anything. The auditor role is that view-only window into the admin side of the operation.

---

## Same-Day Turnovers

**What:** When one booking checks out and a new booking checks in on the same property on the same day, the calendar shows an overlap icon (a double-arrow) on that date. It also counts toward the "same day in and outs" total in the calendar header.

**Why:** Same-day turnovers are the most time-sensitive cleanings in the operation. A cleaner has a fixed window between when one guest leaves and when the next one arrives, often just a few hours. The icon makes these visible at a glance so Admins can prioritize scheduling and cleaners can plan their day accordingly.

---

## Late Checkout and Early Check-In Flags

**What:** Bookings can be flagged with a late checkout time or an early check-in time. These show as icons on both the admin and staff calendars.

**Why:** Late checkouts compress the cleaning window. Early check-ins create urgency. Cleaners need to know before they show up whether they're walking into a tight turnaround or have more flexibility. The flags on the cleaning card give them that information without having to contact the admin.

---

## Check-In / Check-Out Timestamps for Cleaners

**What:** Each cleaning card on the staff dashboard has Check In and Check Out buttons. Tapping them records a timestamp on the booking. The admin calendar reflects the status in real time.

**Why:** Admins can't be at every property. The timestamps tell him where each cleaner is at any given moment — whether they've arrived, how long they've been on site, and when they finished. It's a lightweight time-tracking and accountability system built directly into the cleaning workflow.
                                        