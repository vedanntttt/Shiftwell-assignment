# Shiftwell: Swap Auto-Approval

A clickable prototype for Shiftwell, a fictional shift-scheduling app. Today every shift swap waits for a Location Manager to approve it by hand, which takes a day or more. This prototype shows the fix: Shiftwell checks each swap against a few rules and either approves it instantly or routes it to the right person with a clear reason.

**Live demo:** [shiftwell-assignment.vercel.app](https://shiftwell-assignment.vercel.app)

> All names, locations and data are fictional. "Today" is fixed at **Sun 18 Oct 2026**, so certificate results are always the same.

---

## How it works

1. **Staff requests a swap.** They pick a shift and the colleague who will take it.
2. **Shiftwell runs three checks:** role / training, certificates, and weekly hours.
3. **Approved instantly, or routed with a reason.** Only exceptions reach a human.

On the 20 sample swaps, 12 of 20 (60%) are approved instantly. Today, all 20 would wait for a manager.

---

## Pages

The app is one HTML file with a sidebar (a tab bar on phones). Each page has its own link, for example `#/rules`.

| Page | Link | What it shows |
|---|---|---|
| **Overview** | `#/overview` | The idea in 30 seconds: how-it-works strip, sample-week stats (auto-approved, Location Manager, Regional Approver, blocked), the "today vs with rules" comparison, a shortcut to try a swap, and a summary of the active rules. |
| **New swap** | `#/swap` | The swap request form (location, then shift, then who takes it) and the result: outcome badge, each check with pass / fail / skipped and its reason, and who acts next. On the right are the staff at that location, with the person taking the shift highlighted and their hours before and after the swap. |
| **Rules** | `#/rules` | Admin settings that apply to every location, next to a live **Impact on the sample week** panel that updates as you change any rule. |
| **Staff & shifts** | `#/staff` | One location at a time (FreshBowl or CarePoint): the team, with role, training, hours against the limit and each certificate's status, plus the shifts up for swap. **Start swap** opens New swap with that shift already picked. |
| **Replay** | `#/replay` | Runs 20 sample swaps through the current rules. Shows stat cards, the comparison line and every swap with its outcome. Can be filtered by location. |
| **Decision log** | `#/log` | Every swap checked on New swap, newest first (last 20): time, people, shift, location, outcome and reason. The sidebar shows a count. |

### Why locations are split in some places and combined in others

- **Rules are company-wide.** An admin sets them once, so the Rules page and the overall replay cover both locations.
- **Swaps happen inside one location.** New swap only offers shifts and staff from the chosen location, and Staff & shifts shows one location at a time. The replay can be filtered by location.

---

## Decision logic

Three checks are run for the person taking the shift:

1. **Role / training:** the shift's role is in the taker's "trained for" list.
2. **Certificates:** every certificate the role needs is still valid on the shift date. (Nurses need *BLS* and *Nursing Council Registration*; other roles need none.)
3. **Weekly hours:** hours already this week plus the shift's hours must not exceed the limit.

A check whose rule is switched off is shown as **Skipped**.

**Outcome, by priority.** If several checks fail, the highest-priority outcome wins, but every failed reason is listed.

| Priority | Condition | Outcome | Who acts next |
|---|---|---|---|
| 1 | Auto-approval is off | Sent to Location Manager | Location Manager |
| 2 | A certificate is not valid | **Blocked** | Nobody until it's renewed |
| 3 | Not trained for the role | Sent to Location Manager | Location Manager |
| 4 | Over the weekly hours limit | Sent to the chosen approver (default: Regional Approver) | That approver |
| 5 | All checks pass | **Auto-approved instantly** | Nobody |

Example reasons: *"Kavita Rao's BLS expired on 30 Sep 2026."*, *"Priya Deshmukh is not trained for Line Cook."*, *"Imran Shaikh would reach 49 hrs this week (limit 48)."*

### Rule settings

| Setting | Default | Effect |
|---|---|---|
| Auto-approval | On | Off sends every swap to the Location Manager, which is how Shiftwell works today. |
| Check role / training | On | Off skips the role check. |
| Check certificates | On | Off skips the certificate check. |
| Weekly hours limit | 48 hrs | The cap for hours this week plus the shift. |
| Send over-limit swaps to | Regional Approver | Regional Approver, Location Manager or Admin. |

Every change re-checks the current swap, the replay, the Overview and the Impact panel straight away.

---

## Sample data

**FreshBowl, Koregaon Park (restaurant):** Rahul Patil, Sneha Kulkarni, Imran Shaikh, Priya Deshmukh, Arjun Jadhav. The roles are Line Cook, Grill and Cashier.

**CarePoint, Baner (clinic):** Anjali Menon, Kavita Rao (BLS expired), Deepak Nair (registration expired) and Meera Joshi, all Nurses, plus Rohan Gupta (Receptionist).

**Shifts up for swap:** S1 to S6, three per location. S7 to S13 are used only in the replay.

**Replay mix with default rules:** 12 auto-approved, 3 to Location Manager (role mismatch), 3 to Regional Approver (over 48 hrs), 2 blocked (expired certificate).

### Try these

| Swap | Expected result |
|---|---|
| S1 → Rahul Patil | Auto-approved (45 hrs) |
| S1 → Imran Shaikh | Sent to Regional Approver (49 hrs) |
| S1 → Priya Deshmukh | Sent to Location Manager (not trained) |
| S4 → Kavita Rao | Blocked (BLS expired) |
| S4 → Meera Joshi | Sent to Regional Approver (54 hrs) |

- Set the hours limit to **50**: S1 → Imran becomes Auto-approved.
- Turn **Check certificates** off: S4 → Kavita becomes Auto-approved.
- Turn **Auto-approval** off: everything goes to the Location Manager, and the replay shows 0% instant.

---

## Design

- Calm, minimal B2B dashboard style: a cream-white background (`#FBFAF6`), white cards with thin borders, and a charcoal accent.
- Muted status colours that are always paired with an icon and text: green (auto-approved), amber (Location Manager), blue (over-limit approver), red (blocked).
- Inter font, a short fade on results, and no other animation (none at all for users who turn off motion).
- Accessible: every control has a label, focus rings are visible, and moving between pages puts focus on the page title.
- Responsive down to 375px wide with no sideways scrolling. Below 900px, columns stack and the sidebar becomes a tab bar.

---

## Tech

- **One file:** `index.html`, with inline CSS and plain JavaScript. No framework, build step or backend.
- The only external resource is the Inter font from Google Fonts.
- Navigation uses hash links (`#/page`), so it works when opened straight from disk and on any static host.
- Everything is held in memory, so a page reload resets rules and the log to their defaults.

### Run locally

Double-click `index.html`, or open it in any modern browser.

### Deploy

- **Vercel:** import this repository, set Framework Preset to **Other**, leave the build settings empty and deploy.
- **Netlify Drop:** drag the project folder onto https://app.netlify.com/drop.

### Files

```
index.html          The whole prototype
PROTOTYPE_SPEC.md   The original build spec (describes the earlier single-page layout)
README.md           This file
```
