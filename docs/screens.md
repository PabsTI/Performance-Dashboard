# Screens

[← Back to the README menu](../README.md#menu)

What each screen of the app does, written for the people who use it. The side menu has five options: Files settings, Agent Dashboard, Team Lead (TL) Dashboard, Operation Manager (OM) Dashboard and Account General Manager (AGM) Dashboard. The active one is highlighted.

**Jump to:**

- [Features shared by every screen](#features-shared-by-every-screen)
- [Targets](#targets)
- [Files settings](#files-settings)
- [Agent Dashboard](#agent-dashboard)
- [TL Dashboard](#tl-dashboard)
- [OM Dashboard](#om-dashboard)
- [AGM Dashboard](#agm-dashboard)
- [What is not here (on purpose)](#what-is-not-here-on-purpose)

## Features shared by every screen

- **Header with the role.** Each dashboard's header starts with the person's name and a line that begins with their role: "Agent", "Team Lead (TL)", "Operation Manager (OM)" or "Account General Manager (AGM)", followed by their email and counts.
- **Copy a section as an image.** Every framed section (rounded box) has a copy button in its top-right corner. A tooltip reads "Copy section". Clicking it puts the section on the clipboard as an image, ready to paste into a slide or document. Buttons are left out of the image.
- **Copy the whole screen.** The **Copy screen as image** button at the bottom of the side menu copies the current screen, header and tabs included, without the side menu. On a tabbed dashboard it copies the tab you are on. It works on every screen, Files settings included. On the Agent Dashboard it copies the selected agent's dashboard, not the agent list.
- **If copying is blocked** by the browser, the image is downloaded as a PNG instead and a message says so.
- **Help button.** A "?" button at the bottom of the side menu opens this documentation. It only appears when the `HELP_LINK` script property holds a valid link.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## Targets

Targets live in `CONFIG.TARGETS` (`Config.js`) and come from the original project:

| Metric | Target |
|---|---|
| Quality Assurance (QA) | 90% |
| Overall Satisfaction (OSAT) | 70% |
| Net Promoter Score (NPS) | 65 |
| Agent Satisfaction (ASAT) | 85% |
| Calls at Target (calls with QA on target) | 80% |

> To be confirmed with the account team that they are still current.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## Files settings

**Audience:** whoever runs the weekly upload.
**What it does:** feeds the spreadsheet with the week's data.

- Three fields: QA (`GCDC&C Responses`), Survey (`Raw`) and WFM (`DATA AUX Engine`). They accept the ID or the full URL of the Google Sheet.
- Each field is optional, but at least one is required.
- There is a lock: it must be unlocked to edit, and the **Process & Save to History** button only turns on after a field is modified.
- When done, it shows the rows added per source and warnings about rows with no Roster match. Then it locks again and the IDs are not saved.

**Caveats:**
- It always **appends**, never replaces (except the Roster). Uploading the same file twice duplicates the rows.
- Recommended order: QA, Survey, then WFM, because QA carries the Roster the others need.
- Sources must be Google Sheets, not loose `.xlsx` files.
- The account that deploys the app needs read access to the sources.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## Agent Dashboard

**Audience:** TL or coach, to follow up on an agent.
**How it is used:** search for and pick an agent, then pick a week or "All Time".

Six tabs. Each section has its own copy button (see [features shared by every screen](#features-shared-by-every-screen)); the metric cards and the Key Insight bar form one section.

| Tab | What it shows |
|---|---|
| Overview | QA, OSAT, NPS and ASAT cards against their target with the "Key Insight" bar; trend; distribution; frequency of opportunity codes; "Quick-Win" checklist |
| QA Call Log | Every quality evaluation |
| SMART Goal | Proposed SMART goal |
| 5E Roadmap | Five-step coaching plan |
| Role-Play Log | Manual log of role-play sessions |
| Progress Tracker | Coach notes and clean-call streak |

**AI use:** the "Key Insight" bar, the checklist, the SMART Goal, the 5E Roadmap, the timeline and the projection chart come from the AI. The numbers do not.
**If the AI fails**, the dashboard does not crash. The bar reads "Coaching insight unavailable.", "Primary Target" shows "—" and the other tabs look empty or show generic values. The screen does not show the cause; it is stored in the cache.

**The Clean button.** Next to the failed Key Insight bar there is a small **Clean** button (it does not appear when the insight worked). Its tooltip explains it: the failure was saved, and Clean removes it for this agent and period. Clicking it only deletes the saved failure; it does **not** call the AI. The bar then says "Cleaned. Reopen this view to generate the insight again": select the agent again (and pick the period if needed) and the coaching plan is requested again. Other agents or periods with a saved failure need their own Clean.

**QA opportunity codes** (defined in `CRITERIA_MAP`, `DataAccess.js`):

| Code | Label |
|---|---|
| CF1 | Complete Redemption Options |
| CF2 | Professional Service & Rapport |
| CF3 | Email Confirmation Before Survey |
| CF4 | Website / Survey Offer |
| BF1 | Opening & Closing Scripting |
| BF2 | Documentation Accuracy (UAD) |
| BF3 | Escalation / Transfer Accuracy |
| BF4 | KM Article Utilization |
| BF5 | Tools & Applications Accuracy |
| ComF1 | ID&V / Security Compliance |
| ComF2 | Regulatory Compliance |

> What the letters CF, BF and ComF stand for is not documented in the code; it is unknown.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## TL Dashboard

**Audience:** Team Lead.
**What it shows:** the **aggregated performance of their team**, not the TL as a person.
Pick a TL and a week.

Four tabs: Overview, SMART Goal, 5E Roadmap and Progress Tracker (with a per-agent table). Every block (KPI cards, trend, areas of opportunity, scorecard, roadmap, insights, agent breakdown) is a framed section with its own copy button.
**No AI:** everything is computed locally.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## OM Dashboard

**Audience:** Operation Manager.
**What it shows:** the same as the TL Dashboard, one level up, with a **Team Lead Comparison** table colored against the targets.

- The list of OMs comes from `OM_Roster` (only those with `Active?` = Yes).
- Each OM's TLs come from `TL_Roster`.
- Four tabs: Overview, SMART Goal, 5E Roadmap and Progress Tracker (with the per-agent breakdown and their TL). Every block is a framed section with its own copy button.
- If there are no active OMs, the screen says so ("There are no active Operation Managers.") and shows nothing else.
- **No AI.**

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## AGM Dashboard

**Audience:** Account General Manager.
**What it shows:** the whole account. It opens directly, with no list. The header carries the AGM's name (first active row of `AGM_Roster`) or "Account Overview" if there is none.
A **Period** selector lets you pick a week or "All Time".

Three tabs. Every tab is split into framed sections, each with its own copy button, and the layout is compact so the Overview fits on one screen.

| Tab | Sections (one frame each) |
|---|---|
| Overview | (1) Customer satisfaction (OSAT, ASAT, NPS), Errors and Quality Assurance (QA) evaluations together, in rows of three cards with their target and an *On / Near / Off target* chip, including agents at target; (2) top TL / OM and the ones with the most room to grow; (3) **Week over Week** (selected week against the previous one); (4) the weekly QA trend |
| Supervisors | (1) Operation Manager (OM) Comparison, when there are active OMs (with an "Unassigned" row so totals add up); (2) Team Lead (TL) Comparison, with its colour note |
| Errors | (1) Errors at a glance: total, per evaluation, critical, and by family (CF, BF, ComF); (2) Compliance errors (ComF codes) with the table by TL; (3) Errors by area; (4) Errors by Team Lead (TL); (5) Errors by Operation Manager (OM), when there are OMs |

In the error tables the columns are named "Quality Assurance (QA) Evaluations", "Total Errors", "Errors per Evaluation" and "Critical Errors". The CF, BF and ComF columns keep their short names because the meaning of those letters is not confirmed.

**Definitions specific to this view** (to be confirmed with the account team):
- **Error:** a QA criterion answered "No" on an evaluation.
- **Critical errors:** sum of the `Mistakes with Critical Impact` column.
- **Errors per evaluation:** errors divided by evaluations.
- **Near target:** within 10 points of the target.
- **Agents at Target:** agents with QA at or above the 90 target, out of the agents with data.

**No AI.** The account is the whole active Roster; the leadership tabs only add the OM grouping.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)

## What is not here (on purpose)

These were tried and removed by decision of the project owner: Watchlist, Efficiency (AHT and occupancy), the "Data on file" banner, the Banks tab and the AI Usage page.

[← Back to the README menu](../README.md#menu) · [↑ Screens index](#screens)
