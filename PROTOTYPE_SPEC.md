# Shiftwell Swap Auto-Approval Simulator: Build Spec

Build exactly what this document describes. Read it fully before writing code.

## 1. Purpose

Shiftwell is a (fictional) shift-scheduling app. Today every shift swap waits for a Location Manager to approve it by hand, which takes a day or more. This prototype shows the fix: the app checks each swap against rules and either approves it instantly or routes it to the right person with a reason.

The audience is an interview panel. They should understand the idea within 30 seconds of opening the page.

## 2. Tech constraints

- One file: `index.html` with inline CSS and vanilla JavaScript. No frameworks, no build step, no backend.
- Only external resource allowed: the Inter font from Google Fonts.
- Must work by double-clicking the file, and when dropped onto Netlify Drop.
- Responsive: works at 375px phone width with no horizontal scrolling. Stack columns vertically below 900px.
- Fix "today" in the app as **18 Oct 2026** (a constant) so certificate results are always the same.

## 3. Page layout (top to bottom)

1. **Header**
   - Title: "Shiftwell: Swap Auto-Approval"
   - Subtitle: "Prototype. Swaps that pass the rules are approved instantly. Only exceptions reach a human."
   - Small tag on the right: "Fictional data"

2. **How it works strip**: three small steps in a row (stack on mobile):
   1. Staff requests a swap
   2. Shiftwell checks role, certificates and weekly hours
   3. Approved instantly, or routed with a reason

3. **Main area, three columns on desktop**:
   - Left: **Rules** card (Admin settings)
   - Middle: **Swap request** card, with the **Result** card directly under it
   - Right: **Staff** card (list of staff at the selected location)

4. **Replay section** (full width): "Replay 20 sample swaps" button, summary stat cards, and the list of results.

5. **Decision log** (full width, collapsible): every evaluated swap with time, people, outcome and reason.

6. **Footer**: "Built for the Shiftwell product case. All names and data are fictional."

## 4. Rules card (Admin)

Controls (all changes re-evaluate the current request and the replay instantly):

| Control | Type | Default |
|---|---|---|
| Auto-approval | Toggle | On |
| Check role / training | Toggle | On |
| Check certificates | Toggle | On |
| Weekly hours limit | Number input (hrs) | 48 |
| Send over-limit swaps to | Dropdown: Regional Approver, Location Manager, Admin | Regional Approver |

When **Auto-approval is Off**, every swap goes to "Sent to Location Manager" with the reason "Auto-approval off: manual approval (how Shiftwell works today)". This lets the panel compare today vs the proposal.

## 5. Data

### Role certificate requirements
- Nurse: "BLS" and "Nursing Council Registration"
- All other roles: none

### Locations and staff

**FreshBowl, Koregaon Park (restaurant)**

| Name | Role | Trained for | Hours already this week |
|---|---|---|---|
| Rahul Patil | Line Cook | Line Cook, Grill | 40 |
| Sneha Kulkarni | Line Cook | Line Cook | 32 |
| Imran Shaikh | Grill | Grill, Line Cook | 44 |
| Priya Deshmukh | Cashier | Cashier | 36 |
| Arjun Jadhav | Cashier | Cashier | 24 |

**CarePoint, Baner (clinic)**

| Name | Role | Trained for | Hours this week | BLS expires | Nursing Council Reg expires |
|---|---|---|---|---|---|
| Anjali Menon | Nurse | Nurse | 36 | 10 Mar 2027 | 31 Dec 2027 |
| Kavita Rao | Nurse | Nurse | 30 | 30 Sep 2026 (expired) | 30 Jun 2027 |
| Deepak Nair | Nurse | Nurse | 24 | 15 Jan 2027 | 10 Oct 2026 (expired) |
| Meera Joshi | Nurse | Nurse | 46 | 20 Dec 2026 | 31 Aug 2027 |
| Rohan Gupta | Receptionist | Receptionist | 40 | n/a | n/a |

### Shifts available to swap

| ID | Location | Role needed | When | Hours | Current owner |
|---|---|---|---|---|---|
| S1 | FreshBowl | Line Cook | Tue 20 Oct, 18:00 to 23:00 | 5 | Sneha Kulkarni |
| S2 | FreshBowl | Grill | Wed 21 Oct, 17:00 to 23:00 | 6 | Imran Shaikh |
| S3 | FreshBowl | Cashier | Fri 23 Oct, 12:00 to 20:00 | 8 | Priya Deshmukh |
| S4 | CarePoint | Nurse | Thu 22 Oct, 08:00 to 16:00 | 8 | Anjali Menon |
| S5 | CarePoint | Nurse | Sat 24 Oct, 14:00 to 22:00 | 8 | Meera Joshi |
| S6 | CarePoint | Receptionist | Mon 19 Oct, 09:00 to 17:00 | 8 | Rohan Gupta |

## 6. Swap request card

- Dropdown 1: **Location** (FreshBowl or CarePoint)
- Dropdown 2: **Shift** (only shifts at that location; show "S1 · Line Cook · Tue 20 Oct · 5 hrs · from Sneha")
- Dropdown 3: **Who takes the shift** (staff at that location, excluding the current owner)
- Button: **Check swap**

Default selection on load: FreshBowl, S1, Rahul Patil (so the page opens on a green auto-approval).

The Staff card on the right shows the staff at the selected location with role, hours this week and certificate status (green "Valid" / red "Expired" pill). Highlight the selected taker.

## 7. Decision logic

Evaluate checks for the person taking the shift:

1. **Role check**: shift role is in the taker's "Trained for" list.
2. **Certificate check**: every certificate required for the shift role is valid on the shift date (expiry date on or after the shift date).
3. **Hours check**: taker's hours this week + shift hours is less than or equal to the limit.

Skip a check if its toggle is off (show it as "Skipped" in grey).

Outcome, in this priority order:

| Priority | Condition | Outcome | Colour |
|---|---|---|---|
| 1 | Auto-approval off | Sent to Location Manager | Amber |
| 2 | Certificate check fails | **Blocked** | Red |
| 3 | Role check fails | **Sent to Location Manager** | Amber |
| 4 | Hours check fails | **Sent to [setting]** (default Regional Approver) | Blue |
| 5 | All pass | **Auto-approved instantly** | Green |

If several checks fail, use the highest-priority outcome but list every failed reason.

Reason text examples (use real values):
- "Kavita Rao's BLS expired on 30 Sep 2026."
- "Priya Deshmukh is not trained for Line Cook."
- "Imran Shaikh would reach 49 hrs this week (limit 48)."
- "All checks passed. Approved in 0 seconds, no manager needed."

### Expected results (use these to test)

| Shift | Taker | Expected outcome |
|---|---|---|
| S1 | Rahul Patil | Auto-approved (45 hrs) |
| S1 | Imran Shaikh | Sent to Regional Approver (49 hrs) |
| S1 | Priya Deshmukh | Sent to Location Manager (not trained) |
| S2 | Rahul Patil | Auto-approved (46 hrs) |
| S3 | Arjun Jadhav | Auto-approved (32 hrs) |
| S4 | Kavita Rao | Blocked (BLS expired) |
| S4 | Deepak Nair | Blocked (registration expired) |
| S4 | Meera Joshi | Sent to Regional Approver (54 hrs) |
| S5 | Anjali Menon | Auto-approved (44 hrs) |

## 8. Result card

- Large outcome badge with icon and colour (from the table above).
- Checklist of the three checks, each with a pass/fail/skipped icon and a one-line reason.
- One line: "Who acts next: nobody" (auto-approved) or the role it was routed to.
- Animate in with a short fade (150 to 200ms). No other animation.

## 9. Replay section

Button: **Replay 20 sample swaps**.

Define 20 sample swaps from the data above, evaluated independently (hours do not add up across samples). Use this exact mix with the default rules:

- 12 auto-approved
- 3 sent to Location Manager (role mismatch)
- 3 sent to Regional Approver (over 48 hrs)
- 2 blocked (expired certificate)

Show four stat cards: % auto-approved (60%), % to Location Manager, % to Regional Approver, % blocked. Under them, a compact list of all 20 (taker, shift, outcome pill). On mobile the list becomes stacked cards, never a wide table.

Add one comparison line under the stats:
"Today: 20 of 20 swaps wait for a manager. With rules: 12 of 20 approved instantly, 8 need a human."

Update the line live when rules change (for example, turning Auto-approval off shows 0 approved instantly).

## 10. Decision log

Each "Check swap" click adds a row at the top: time, taker, shift, outcome pill, reason. Max 20 rows. Collapsible section, collapsed by default on mobile.

## 11. Visual style

**Feel:** clean B2B SaaS dashboard. Calm, lots of white space, nothing flashy.

**Font:** Inter (400, 500, 600, 700). Base size 15px; headings 600.

**Colours (define as CSS variables):**

| Token | Hex | Use |
|---|---|---|
| --bg | #F6F7F9 | Page background |
| --card | #FFFFFF | Cards |
| --border | #E5E7EB | Card borders, dividers |
| --text | #111827 | Main text |
| --muted | #6B7280 | Secondary text |
| --primary | #4F46E5 | Buttons, toggles, focus rings |
| --success | #16A34A | Auto-approved (bg #DCFCE7) |
| --warning | #D97706 | Sent to Location Manager (bg #FEF3C7) |
| --info | #2563EB | Sent to Regional Approver (bg #DBEAFE) |
| --danger | #DC2626 | Blocked (bg #FEE2E2) |

**Components:**
- Cards: white, 1px border, 12px radius, 20px padding, very soft shadow (0 1px 2px rgba(0,0,0,0.04)).
- Buttons: primary filled indigo, 8px radius, 40px tall; hover slightly darker.
- Toggles: pill switches, indigo when on.
- Outcome pills: rounded-full, tinted background with the strong colour as text.
- Max content width 1200px, centred, 24px gutters (16px on mobile).
- Icons: simple inline SVG or Unicode (check, cross, arrow, clock). No icon libraries.

**Accessibility:** labels on every control, visible focus ring, colour is never the only signal (always text plus icon).

## 12. Done when

- Every row in the "Expected results" table produces the stated outcome.
- Changing the hours limit to 50 turns S1 + Imran into Auto-approved.
- Turning Certificate check off turns S4 + Kavita into Auto-approved.
- Turning Auto-approval off sends everything to Location Manager and the replay shows 0% instant.
- No horizontal scroll at 375px width; no console errors.

## 13. Deploy

Drag the project folder onto https://app.netlify.com/drop to get a public link.
