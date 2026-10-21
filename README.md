# Artifactory Onboarding Dashboard

A customer-facing, interactive tracker for the JFrog Artifactory **60-day onboarding plan**. It is run by the Customer Success (CSM) team and shared with each customer. It runs in two modes from the same file:

- **Standalone (current primary use):** open `dashboard.html` in a browser or host it anywhere. Progress is saved in the browser and shared by exporting and importing a JSON file. See "Using it without claude.ai".
- **claude.ai artifact:** https://claude.ai/artifact/5oj5E1b5Mn4fhUESEJYEMP (private until shared from the artifact's Share menu). Progress is shared live through the artifact database. The published copy is the earlier version, without the export/import bar.

## Files in this folder

| File | Purpose |
|---|---|
| `dashboard.html` | The dashboard: one self-contained file with CSS, JS and plan content. Edit this. |
| `JFrog-New-Logo-Adoption-30_60-Plan.odt` | Original DSE-facing plan. Source for weekly content. Only the 60-day plan was used. |
| `JFrog_Onboarding_Guide_Template.pdf` | Export of the Notion onboarding guide. Source for resource links. |
| `notion-link.txt` | URL of the original Notion page (it is JS-rendered, so it can't be fetched directly). |

## What it does

Tabs: **Journey** (8-week stepper with goal, agenda, homework checklist, blockers and resources per week), **Milestones**, **Repositories**, **Success criteria** (plus the post-Day-60 monthly check-in), **Issues**, **Resources and guides**.

The plan adapts to account settings (Resources tab, "Account settings"; in the artifact version only editors see it):
- **Customer name**, **CSM name** and **start date** fill the header. The start date also shows the Day 60 target.
- **Deployment** on-prem / SaaS / hybrid shows or hides install, HA and SaaS-specific steps.
- **Curation** and **JAS** purchased flags show or hide add-on steps. Week 5 is skipped if neither is purchased.

### Changing the company name
1. Open the **Resources and guides** tab.
2. Edit **Customer name** in the Account settings card, then click outside the field. The header updates to "<name> onboarding".
3. Standalone mode stores it in that browser only. Use **Export progress** to pass it on.

To change the default name shown before anyone edits it, change `customer` in `const DEF={...}` in `dashboard.html`. Browsers that already saved a name keep theirs.

## How content maps to the sources

- ODT drives what happens each week (agenda, homework, blockers, milestones, success criteria).
- Notion PDF drives where to learn it (the `LINKS` block).
- Rewritten for CSMs: "DSE" becomes "your JFrog CSM / JFrog team", and "red flags: escalate" becomes customer-visible "Common blockers".
- **Deliberately excluded (internal only):** capacity and hiring model, automation roadmap, Director escalation templates, Salesforce HUD stage updates, deal size and churn fields, the National Defence appendix, reviewer comments, the 30-day plan, and Notion pages 7-8 (internal notes, mailboxes, named staff).

## Editing the dashboard

All in `dashboard.html`:
- **Weekly content:** the `WEEKS` array. Items can be plain strings or `T('text', tag)` where tag is `onprem`, `saas`, `curation` or `jas` to limit them to a track or add-on.
- **Links:** the `LINKS` block. A `null` (`SOON`) URL renders as "link coming soon". **Still to fill:** Slack channel, feedback survey, progress tracker, Xray / Curation / JAS guides, CircleCI, TeamCity, Bitbucket Pipelines, AWS CodeArtifact.
- **Milestones / scorecard / guides:** `MILESTONES`, `MUST`, `NICE`, `GUIDES`.
- **Optional migration card:** the `<details class="opt">` block in `renderJourney` (Weeks 1-2) plus the `mig` entry in `GUIDES`. Remove both to drop it.
- **Theme:** tokens at the top of the `<style>` block (neutral JFrog green, light and dark). No logos used.
- Republish by passing the same file to the Artifact tool with `url` set to the live artifact, so the link is kept.

## Using it without claude.ai (standalone mode)

`dashboard.html` is a standalone file. Open it directly in a browser or host it on any static host (GitHub Pages, S3, SharePoint, an internal web server). No build step.

When claude.ai storage is not available, it automatically switches to **local mode**:
- Progress is saved in that browser only (`localStorage`, key `jfrog-onboarding-v1`). Clearing site data loses it.
- Everyone has edit rights in their own copy, since there is no sign-in. Account settings (Resources tab) are open to whoever uses the file.
- A blue bar offers **Export progress** (JSON file), **Copy as text**, **Import file**, **Paste instead** and **Undo import**.
- Import checks the file: only known sections are accepted, unsafe keys and invalid values are rejected or reset, and the previous progress can be restored with Undo import.
- Fonts load from Google Fonts and fall back to system fonts if blocked.

Typical flow:
1. The CSM opens the file, sets customer, deployment, add-ons and start date in Account settings, then exports.
2. The CSM sends the customer the HTML file together with the exported JSON. The customer imports the JSON.
3. The customer ticks homework, then exports and sends the JSON back. The CSM imports it to see progress.

Limitations: no live sharing, no editor-only controls, and two people editing in parallel need to merge by hand (last import wins).

Publishing to claude.ai: the file starts with a doctype and meta tags for standalone use. Remove its first three lines before passing it to the Artifact tool, which adds its own skeleton. The already-published artifact is the earlier version without local mode.

## Data and permissions (claude.ai artifact mode)

Progress is stored in the artifact's shared database (`db` capability), so the CSM and customer see the same state.

| Data | Where | Who can write |
|---|---|---|
| Homework checkboxes | `onboarding/checks` | Contributor and above |
| Repositories | `repos/*` | Contributor and above |
| Issues | `issues/*` | Contributor and above |
| Account config | `onboarding/config` | Editor/Owner only |
| Week status | `onboarding/weeks` | Editor/Owner only |
| Milestones | `onboarding/milestones` | Editor/Owner only |
| Success scorecard | `onboarding/scorecard` | Editor/Owner only |

Viewers and commenters get a read-only view. Anyone who needs to tick boxes must be given **Contributor** access in the Share menu.

## Per-customer setup

State is per artifact, so use **one copy per customer**:
1. Publish a new artifact from `dashboard.html` (no `url`, so a new link is created) with `capabilities` of `db` (rules below) and `user`.
2. Set customer name, CSM, deployment, add-ons and start date in Account settings.
3. Remove the three example repos and share the link with Contributor access.

Rules to declare: `onboarding/config`, `onboarding/weeks`, `onboarding/milestones`, `onboarding/scorecard` each with `write: "admin"`.

Currently seeded with placeholder data ("Acme Corp (example)" and three example repos). Replace before sharing.

## Known gaps and notes

- Source Notion guide has an unfinished Xray / JAS / Curation section, so those links are placeholders.
- The Notion "support@jfrog.com" link points to `service@jfrog.com`. Confirm the correct address before adding one.
- Not yet checked in a browser: page layout at phone width, light and dark themes, and the export/import buttons. Checked so far: script syntax, the artifact database access rules, and a scripted test of the local save and import logic (round trip, bad files rejected, config values reset).
- Standalone mode has no live sharing. If that becomes a need, the next step is a shared backend (for example Firebase or Supabase) in place of `makeLocalDb()`, which exposes the same small interface as the artifact database.
- Viewer access levels are not strict: keep CSM-only notes out of this page.
