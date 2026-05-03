Build a single self-contained HTML demo file (no frameworks, no libraries — plain HTML + CSS + vanilla JS) for a workflow application view page with the following spec:

---

## Tabs

Three tabs at the top. **Reviewer view** and **Approver view** tabs are hidden on load. They only appear after the first submission is made on the First submission tab.

1. **First submission** — always visible
2. **Reviewer view** — hidden until first submission
3. **Approver view** — hidden until first submission

---

## Layout

- Sticky top nav bar with brand name and breadcrumb (Applications > View application)
- Tab switcher below the nav
- Single expansion panel (chevron collapses/expands body with smooth animation) per tab
- Max content width 720px centered on a light grey page background

---

## Panel Header (always visible)

```
Left:   [Month Year] Package     e.g. "May 2026 Package"
Right:  [Status badge]  [Action buttons]
Far right: Chevron
```

---

## Status Badges

Text only — no dot or icon inside. Pill-shaped with coloured background.

| Status | Style |
|---|---|
| Open | Grey background, dark grey text |
| Submitted | Amber background, amber text |
| Locked | Green background, green text |
| Open - Sent for revision | Red background, red text (activity log only) |

---

## Tab 1 — First Submission

Shows the initial **Open** state. Panel body is empty (no sections).

- Blue info banner above the panel explaining this tab is for the initial submission demo, and that Reviewer/Approver view tabs are pre-populated
- Status badge: **Open**
- Button: **Submit for review** (solid blue, enabled)
- Panel body: empty initially

### On clicking Submit for review
- Modal opens: **"Submit May 2026 Package for approval"**
- After confirming submit:
  - Panel body shows a success message: "✓ Submitted successfully. Switch to Reviewer view or Approver view to see the post-submission screens."
  - Status badge → **Submitted**
  - Submit for review button → disabled
  - **Reviewer view** and **Approver view** tabs become visible

---

## Tab 2 — Reviewer View

Pre-populated on load with mock post-submission data. Does not depend on Tab 1 action.

**Shared state with Approver view** — any action in one reflects in the other instantly.

### Button states

| Status | "Submit for review" |
|---|---|
| Open | Enabled — solid blue filled |
| Submitted | Disabled |
| Locked | Disabled |

### Pre-populated mock data
- Status: **Submitted**
- Supporting documentation: a sample PDF file with name, size, uploader and timestamp
- Activity log: one entry — Sarah Mitchell | Submitted | sample comment | 1 May 2026, 09:14 AM

### Panel body sections (always visible in this tab)
In this order, separated by dividers:
1. Supporting documentation
2. Activity log

---

## Tab 3 — Approver View

Pre-populated on load with the same mock post-submission data. Shared state with Reviewer view.

### Button states

| Status | "Send back for review" | "Lock Month" |
|---|---|---|
| Open | Disabled | Disabled |
| Submitted | Enabled — solid red filled | Enabled — solid blue filled |
| Locked | Disabled | Disabled |

### Panel body sections (always visible in this tab)
In this order, separated by dividers:
1. Send back textarea area (hidden by default, appears above supporting docs when triggered)
2. Supporting documentation
3. Activity log

---

## Supporting Documentation Section

- Section label: "SUPPORTING DOCUMENTATION" (small caps, muted, monospace font)
- File row: PDF icon on left | file name + meta (size · uploaded by name · timestamp) in middle | download icon button on right (turns blue on hover)
- Note below: *"Only the latest file is stored. Previous versions are no longer available."* (italic, muted)
- Updates to latest file on each reviewer resubmission
- Same file shown in both Reviewer and Approver views

---

## Activity Log Section

- Section label: "ACTIVITY LOG" (small caps, muted, monospace font)
- Descending order — newest entry at top
- Latest entry has a subtle background tint
- Entries are append-only
- Same entries in both Reviewer and Approver views

### Each entry layout
```
[DD Mon YYYY, HH:MM AM/PM]  |  [Author Full Name]        [Status badge]
Comment text…
```

---

## Reviewer Submit / Resubmit Modal

Triggered by "Submit for review" on Reviewer view tab.

- **Title**: "Re-submit May 2026 Package for approval"
- **Comment** *(mandatory \*)* — editable textarea, pre-filled with: "Reviewed all statements and submitting for review with additional documentation."
- **File attachment** *(mandatory \*)* — dashed border upload area with upload icon button on the right. Turns green with checkmark icon when file selected. Accepted: PDF, DOCX, XLSX up to 20MB.
- **Submit request button** — disabled until both comment AND file are provided
- **Cancel** — closes modal, resets upload state
- Clicking outside the modal closes it

### On Submit
- Prepends new entry to both logs (Author: Sarah Mitchell, Badge: Submitted)
- Updates supporting documentation file info in both views
- Status → **Submitted** on both views
- "Submit for review" → disabled
- Approver "Send back for review" and "Lock Month" → enabled
- Modal comment resets to pre-fill text for next time

---

## Approver Send Back Flow

Triggered by "Send back for review".

- Button hides from header
- Inline textarea slides open inside panel body, above supporting documentation section
- Warm amber tinted background on the textarea area
- Label: "Reason for sending back"
- **Submit request** button below — disabled until comment typed
- **Cancel** button — closes textarea, restores button

### On Submit request
- Textarea closes; "Send back for review" reappears but stays disabled
- Prepends new entry to both views (Author: James Crawford, Badge: Open - Sent for revision)
- Status → **Open** on both views
- Both approver buttons → disabled
- Reviewer "Submit for review" → enabled

---

## Lock Month Flow

Triggered by "Lock Month".

### Confirmation dialog
- **Title**: "Lock May 2026 Package"
- **Message**: "Are you sure you want to lock the package for May 2026? The changes can't be undone!"
- **Buttons**: Cancel | Lock Month (solid blue)
- Clicking outside the dialog closes it (same as Cancel)

### On Confirm
- Prepends new entry to both views (Author: James Crawford, Badge: Locked)
- Status → **Locked** on both views
- All buttons across both views → disabled permanently

---

## Fonts

Google Fonts: **DM Sans** (body) and **DM Mono** (timestamps, section labels).

---

## Colours

```
--bg:         #F4F5F7
--surface:    #FFFFFF
--border:     #E2E4E9
--text-pri:   #111827
--text-sec:   #4B5563
--text-muted: #9CA3AF
--amber-bg:   #FEF3C7   --amber-txt: #92400E
--green-bg:   #D1FAE5   --green-txt: #065F46   --green-dot: #059669
--blue-bg:    #DBEAFE   --blue-txt:  #1E40AF   --blue-dot:  #3B82F6
--red-bg:     #FEE2E2   --red-txt:   #991B1B   --red-dot:   #EF4444
--open-bg:    #F3F4F6   --open-txt:  #374151
```
