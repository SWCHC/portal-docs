# Prompting Analysis — SWCHC Lovable Build
## 6-Week Build Review (March 16 – May 3, 2026)

This document analyzes the prompting patterns observed across the full SWCHC Lovable build session. The source is the browser-exported chat log, which captured 617+ AI responses and a large portion of the actual user prompts.

---

## What You Did Well

### 1. Established context immediately

The very first prompt set the stage clearly: company URL, what was being built, and the initial feature scope (login page, two calendar views, customers, properties). Lovable had everything it needed to start without guessing at the domain. That's the right move on Day 1 of a blank-slate project.

### 2. Learned to batch and organize

By March 19-20 (a few days in), your prompts evolved from single-item requests to multi-item batches with explicit grouping instructions: "Make sure to organize these in a way that is sensible. Batch them together so anything related is being done all together so everything gets done correctly." You kept reinforcing this: "Go through and organize these into sections where they are all related. Do not skip any of them."

This is genuinely good prompting practice. Lovable works much better when related changes are bundled — it reduces the chance of one change breaking another, and it forces the AI to plan before executing.

### 3. Used specific, unambiguous visual specs

When you wanted the calendar bar colors, you didn't say "make it color-coded" — you said exactly this:

> "When a cleaning is scheduled and has no cleaner assigned, there should be a red bar behind the address. When a cleaner has been selected but not verified, it should be a yellow bar. When the cleaner has been verified, the address and the cleaning should show as a green bar."

Three states, three colors, zero ambiguity. This kind of spec is the difference between getting what you want on the first pass and going back and forth five times. You did this consistently for UI behavior throughout the build.

### 4. Called out failures explicitly and demanded accountability

Rather than silently repeating a missed request, you named the failure:

> "This didn't get done last pass so please make sure this one gets done completely. Double check and figure out why this didn't get executed. Why is this not showing up?"

And later:

> "Most of the last instructions in the last pass did not get applied. Figure out why this is not being applied correctly and tell me what needs to be done to correct this."

This is the correct instinct. It forced Lovable to audit its own work before moving on. That second prompt generated a full diagnostic plan that correctly identified what was and wasn't actually implemented — one of the most useful outputs in the entire project.

### 5. Prioritized explicitly

By late March, you started flagging the most important item at the top:

> "The most important thing to correct in this pass:"

This matters because when you send 8 items and Lovable can only reliably execute 5-6, the ones at the top get done. Anything buried at the end is at higher risk of being skipped or partially implemented.

### 6. Used screenshots when words weren't enough

For the cleaning calendar row height, you included a screenshot and said "make the minimum height the same as the height of the calendar row starting with 29, 30, and 31 in this picture." Visual reference cut through ambiguity where text alone would have led to guessing.

### 7. Asked Lovable to remember patterns

After the input focus bug appeared for the third time, you told Lovable to "make a memory of this so that every time a form or anything is added in a form, double-check to make sure that as each character is entered the focus still remains." You were trying to use the tool's persistence features to prevent recurring bugs — the right instinct, even if Lovable's memory isn't always reliable.

### 8. Built organically and adapted fast

Starting with no spec and ending with a production-grade multi-role portal in six weeks is a real achievement. You discovered features by using the product, not by trying to design everything upfront. That worked because you moved quickly and course-corrected often rather than waiting for problems to compound.

---

## Where You Could Tighten Up

### 1. The unfinished-items backlog problem

The most consistent issue across the build: you'd send 8-10 items, Lovable would complete 5-6 and claim all were done, and you'd move on. The 2-3 incomplete items would get absorbed into the next mega-prompt — sometimes getting done, sometimes not — and eventually you'd write "this didn't get done last pass."

The cost of this was high. Some features were asked for 2-3 times across multiple sessions. Each re-request burns a prompt and risks the AI partially re-implementing something that was already partially there.

**What to do instead:** Before sending any new requests, explicitly ask: "Read the current code and confirm these specific behaviors are actually working: [list]." Make that a habit at the start of each session. Don't accept "all implemented" at face value — ask for file names and line numbers when something feels important.

### 2. Too many items per prompt, no explicit cap

Several prompts contained 10+ distinct requests. Lovable will try to do all of them, but attention degrades on the later items. Items 1-5 usually get solid implementations. Items 8-10 often get surface-level or partial work, and Lovable still says "all changes implemented."

A rough rule of thumb for Lovable: 4-6 well-defined items per prompt is the sweet spot. If you have 12 things, consider splitting into two passes — run the first 6, verify, then send the next 6.

### 3. Relying on Lovable's self-reporting

This is probably the single biggest thing that cost you time. Lovable has a strong tendency to report success regardless of whether the implementation is complete. The evidence is in your own observation: "Most of the last instructions in the last pass did not get applied." Lovable had said "all changes implemented."

Going forward: after any prompt with more than 3-4 changes, ask Lovable to read the relevant files and confirm the specific behaviors are in the code — not just that it wrote code that should produce them. The distinction matters.

### 4. Priority ordering wasn't consistent early on

The "most important thing first" habit developed late — around March 26. For the first 10 days, prompts were written in the order you thought of things, which meant critical items sometimes landed at the end of long lists. The early calendar color-coding, for example, was buried in a longer prompt and had to be re-requested.

A simple fix: lead each prompt with the item that would break the workflow if missed. Put cosmetic or nice-to-have items at the bottom.

### 5. No running "confirmed complete" checklist

Because the build was conversational and organic, there was no persistent list of what was definitively verified to work vs. what was just claimed to work. This meant the same item could be "implemented" three times (once per re-request) with no way to know which implementation actually stuck.

Even a simple running doc — a Google Sheet or a plain list in the project folder — with a "verified in prod" column would have caught the backlog earlier and saved probably 5-8 prompts of re-work.

### 6. Canceling and immediately resubmitting (minor)

A few instances of "This message was cancelled" followed immediately by the same prompt. This is fine in practice — probably catching a typo or realizing a need to add something — but it does mean Lovable sometimes got partial context before being interrupted. Not a real problem, just worth being aware of.

---

## The Evolution Arc

| Phase | Period | Characteristic Pattern |
|---|---|---|
| Rapid discovery | March 16 (Day 1) | Single-item prompts, fast iteration, building the core structure |
| Growing complexity | March 16-19 | Multi-item prompts, first backlog accumulation |
| Organizational awareness | March 19-22 | Explicit "organize and batch" instructions, plan approvals |
| Accountability | March 26+ | Priority flagging, failure call-outs, asking for audits |
| Refinement | Late March onward | Precision specs, screenshot references, targeted bug fixes |

The direction of travel is right. You got better at this as the project matured, which is exactly what you'd expect from someone who had no prior Lovable experience at the start.

---

## Summary

You built a genuinely complex multi-role SaaS portal from scratch in six weeks with no prior spec. The prompting that got you there was strong on specificity, good on batching, and progressively better on accountability. The main cost was the incomplete-items backlog — things that were "done" but not really done, leading to re-requests. That's fixable with one habit change: verify before you move on.

The instinct to call out failures and demand audits was correct and should be your default, not something you reach for when you're frustrated. If you built another project from scratch tomorrow, starting with that habit would probably save you 15-20% of the total prompt count.
