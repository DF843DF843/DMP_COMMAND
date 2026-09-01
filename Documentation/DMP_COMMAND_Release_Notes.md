# DMP COMMAND - Release Notes

Automatisch aus der In-App Release-Notes-Seite (scrReleaseNotes.pa.yaml) exportiert. Diese Datei wird ab sofort bei jedem neuen App-Release aktualisiert (KI-Regel vom 2026-09-01).

## App Changes

### v1.22.0 - 2026-09-01 (current)

- Typography aligned with the official Eurex Brand Manual (page 39/40) - the manual itself specifies Arial as the correct system-compatible substitute for its Replica/Noto Sans brand fonts on platforms without embedded custom fonts, which is exactly our situation (a browser-based Power App)
- Every text across all 9 screens switched from Segoe UI to Arial (367 formula updates), a purely visual change - no layout, colors, or functionality affected
- All future new text formulas should use Font.'Arial' going forward

### v1.21.0 - 2026-09-01

- Refresh indicator (gray on very first load, yellow blinking on every later refresh) extended to all reloading displays across the app - the System Health legend popup dots and all 6 Agent Monitoring status dots now follow the same convention as the Files panel and Automation Status
- Emails Processed colors reassigned to match the requested convention - Blue=No DMP, Green=Not Effected, Purple=Internal, Red=External
- Maintenance Domains - all View/Edit/Replace buttons shifted slightly left so Replace no longer sits almost flush with the card's right edge
- Audit Trail - Open full Audit Trail file button made smaller (less empty space around the text)
- Note - the Audit Trail button still opens the raw Excel file (triggers a browser Save dialog); switching it to a proper SharePoint view will happen together with the already-planned Audit Trail SharePoint migration, not as a standalone guess

### v1.20.0 - 2026-09-01

- Help / Operational Manual redesigned with the same version-menu pattern as Release Notes - a small section menu on the left (About this manual, Colour/LED legend, Top bar, Operating State, Maintenance Domains, Files, System Health, Emails Processed, Automation Status, Next Steps, Sidebar navigation), one large readable container on the right
- The right side is now split in two - a screenshot area on the left (currently a placeholder until real screenshots are provided) and the description text on the right, as requested
- No content was changed, only the layout - all existing section texts are unchanged

### v1.19.0 - 2026-09-01

- Release Notes redesigned - a small version menu now sits on the left (click any version to view it), with one large, fully readable container on the right showing the selected version's full description, for both the App Changes and Agent (Flow) Changes tabs
- Fixed the Copy to Clipboard button, which had not been updated since around v1.11.0 and was silently missing nine newer versions - it now always copies the complete history of the currently selected tab
- The previous stacked-cards layout still exists underneath (hidden) as the data source for the new menu - no release history was lost in the redesign

### v1.18.0 - 2026-09-01

- LED refresh behaviour further refined per feedback - gray only on the very first app load, yellow blinking on every later refresh (kept from v1.17.1)
- Colors aligned with the official Eurex Brand Manual - the Emails Processed donut now uses the official secondary/tertiary colors (Light Blue, Purple, Aqua Mint, Peach) instead of two non-brand shades
- The DMP/SIMU (Operating State) toggle's active color, its status text, and the four screen-edge frame bars now use Eurex Purple for the SIMU+DMP combination instead of a non-brand orange
- Genuine danger/safety colors (PROD environment, PROD+DMP combination) intentionally left as-is - the Eurex palette has no red, and a universal danger signal was judged more important than brand purity there
- All future color and text/typography decisions will reference the official Eurex Brand Manual document

### v1.17.1 - 2026-09-01

- Hotfix for a v1.17.0 import error (PA1001, YamlInvalidSyntax) - two lines in scrHome.pa.yaml had drifted one space out of indentation during an earlier automated edit, which pac canvas pack does not catch but Studio's import does
- LED refresh indicator refined per feedback - LEDs now show gray only during the very first app load (before any data has ever arrived); on every later refresh they blink yellow instead of turning gray, keeping the last known Green/Red status visible in between blinks
- No other visual changes in this release

### v1.17.0 - 2026-09-01

- Fixed Timer/LED freeze during Replace - the file-replace flow call ran inside the same 1-second ticking timer as the auto-refresh, blocking its blink-phase and countdown for the whole call. Moved to its own dedicated timer
- Automation Status column labels widened further, Agent 4 Live Status name no longer clips
- Agent Monitoring now force-refreshes the Agent Status SharePoint list every time the screen is opened, so changes made by a Replace or any agent run show up immediately instead of a stale cached copy
- Now button and the periodic auto-refresh timer now also force-refresh the Agent Status, Configuration and Internal Domains SharePoint lists (previously only Agent Monitoring's own screen open did this)
- System Health and Emails Processed wheels show REFRESHING... while a refresh is running
- All Files-panel and Automation Status LEDs now turn gray while refreshing, for a consistent refreshing indicator
- Admin Functions - Delete button moved below the text and left-aligned like No DMP, text shortened to Delete, section headers restyled to match the Cockpit style (green, uppercase)

### v1.16.1 - 2026-09-01

- Hotfix for a v1.16.0 regression - switching the SIMU/DMP or PROD/SIMU toggle no longer called its flow and the LED stayed green
- Root cause - the new background Reset() calls (Now button, periodic auto-update) ran unconditionally, even while a real toggle click's own flow call was still in flight - this could yank the toggle back to its old value mid-click and made the click appear to do nothing
- Fix - those background Reset() calls now only run when no toggle click is currently being processed
- No visual changes in this release

### v1.16.0 - 2026-09-02

- Fixed Operating State inconsistency - the header text and the Mode/Application toggles could show contradicting values (e.g. header "SIMULATION" while toggle showed "Normal"). Root cause - only the initial screen load resynced the toggles from the refreshed data; clicking "Now" or waiting for the periodic auto-update refreshed the underlying values but never told the toggle controls to redraw, since a Toggle's Default property only applies on first render
- Added an explicit Reset() of both toggles at all four places the operating mode is refreshed (initial load, kickstart timer, Now button, periodic auto-update) so the toggles always redraw in sync with the header
- No other visual changes in this release

### v1.15.0 - 2026-09-01

- Timer stability fix - removed a redundant manual Reset() on the auto-refresh timer that was fighting with the timer's own AutoStart mechanism, likely causing it to stop and not restart after navigating away from the Cockpit (e.g. clicking a KPI tile)
- Maintenance Domains - widened the count numbers further (still clipping to "1..'' for two-digit counts)
- Automation Status - widened the Agent 4 Live Status name column (was clipping to "Agent 4 Live Stat…")
- Admin Functions - widened the mailbox cleanup description so it no longer clips

### v1.14.0 - 2026-09-01

- Debug status bar turned back off (served its purpose - confirmed no Agent 4 connection error)
- Release Notes - "Agent (Flow) Changes" tab header no longer wraps; fuller version history added for Agents 1/3/5
- Maintenance Domains - INTERNAL/EXTERNAL labels shifted right so the count numbers display correctly again
- Automation Status redesigned to match the Files panel style (SOURCE/ACTIVE columns instead of one long line per item)
- Audit Trail (Detail) - fixed the clipped "Warnings" label and the wrapping "Open full Audit Trail file" button
- Admin Functions - removed Display Diagnostics (rarely needed, added clutter); Mailbox cleanup button now left-aligned next to its label instead of pushed to the far right

### v1.13.1 - 2026-09-01

- Hotfix - v1.13.0 could not be opened in Studio (a "Wrap" setting was mistakenly added to several buttons, which only Labels support, plus one line had drifted out of its correct indentation) - both fixed, no visible/functional change
- Audit Trail (Detail) page filled in - shows the 10 most recent Critical and 10 most recent Warning rows read live from the central Audit Trail, plus a link to open the full file (Agent 4 v1.4.0 now also reads and returns these rows)

### v1.13.0 - 2026-09-01

- Debug status bar temporarily made visible again on the Cockpit to help diagnose a reported refresh issue - please report back what it shows
- Sidebar - "Admin Functions (Test Only)" renamed to plain "Admin Functions"
- Maintenance Domains - the separate blinking LED next to Replace was removed; the leftmost status dot on the External row now does the slow blink itself while a replacement upload is processing
- Maintenance Domains - Internal/External rows reflowed with more room for the INTERNAL/EXTERNAL labels so they no longer get cut off with "..."
- Agent Monitoring - fixed the green agent name headings having their descenders (e.g. the letter g) clipped off
- Help / Operational Manual - added a proper scrollbar to the section list (it was missing an explicit height, so long pages were not fully readable)
- Maintenance page - fixed clipped description text in Connection Diagnostics and Versions; the app version is now a clickable link to Release Notes; Admin portal link buttons no longer wrap their text
- Admin Functions - each action (mailbox cleanup, counter reset) now has its own bordered sub-section; Display Diagnostics made smaller and less prominent; the "Internal Sender" reset button no longer wraps its text; remaining "Test"/"Test Only" wording removed
- Cockpit - clicking the CRITICAL or WARNINGS KPI number/label now also opens Audit Trail (Detail); clicking AGENTS ACTIVE now also opens Agent Monitoring - both still acknowledge the change-notification LED as before

### v1.12.1 - 2026-09-01

- Hotfix - v1.12.0 could not be opened or published in Studio (YAML parsing error on 3 new labels that had a colon immediately followed by a space inside the quoted text, e.g. the words "Active rows" followed directly by a colon and a space). Rewritten so the colon is now the very last character of its own quoted segment - parses cleanly, no visible or functional change.

### v1.12.0 - 2026-09-01

- Help / Operational Manual page filled in - a full container-by-container explanation of the Cockpit (what every tile shows, what every button does, and what the colours/LEDs mean), reachable from the sidebar
- Sidebar renamed "Operational Board" to "Help / Operational Manual"
- Configuration (Lists) page filled in - direct View/New links and live active-row counts for all 3 SharePoint configuration lists
- Maintenance page filled in - live connection diagnostics for the 3 SharePoint lists, app/agent version overview, and quick links to the Power Automate, SharePoint and Power Apps maker portals
- Admin Functions - added counter reset actions (No DMP / Internal Sender / External / Not Effected / Reset ALL), each with a confirmation step; the previous value is written to the Audit Trail as an Operational History record before the reset (Agent 6 v1.2.0)

### v1.11.0 - 2026-09-01

- New - direct SharePoint connections added for "DMP Command Configuration", "DMP Command Agent Status" and the brand-new "DMP Command Internal Domains" list (Title/Active columns)
- Internal Domains management now reads/links directly to the new SharePoint list (count + View/Edit buttons) instead of the old flat text file - backend agents still write the text file for now, that migration is planned separately
- Automation Status redesigned as a compact 4-line panel (Agent 4 Live Status, Internal Domains active count, Config Parameters active count, Agent Status entries count), moved into the former "Reserved for future use" tile
- Removed the old separate Automation Status checklist section (generic manual next-step items) to make room - see backlog for the removed items
- All container gaps unified to a single, minimal 8px spacing (previously a mix of 8/16px)
- Maintenance Domains - removed the extra dashes around INTERNAL/EXTERNAL, widened the labels, narrowed the card to match Operating State's width
- Files card narrowed further, tighter column spacing
- System Health / Emails Processed rings resized so their combined card height lines up exactly with Operating State + Maintenance Domains
- System Health ring - the plain-text tooltip was unreliable (sometimes showed nothing at all), replaced with the same click-to-open colour legend popup already used on Emails Processed
- Release Notes list scrollbar fix - it needed an explicit calculated height, not just automatic space-sharing, to actually start scrolling
- Fixed a recurring backend issue where the Agent 4 connection had to be manually deleted and re-added after almost every deployment - traced to a stale internal reference that is now kept in sync
- v1.10.1 - Release Notes cards still cut off long text - AutoHeight alone was not reliable, so each card now has its own fixed max height with a scrollbar (in addition to the outer list scrollbar)
- v1.10.1 - Automation Status - fixed the third bullet (Config Parameters) showing no text at all - the Active column in DMP Command Configuration is a Choice field, needed .Value
- v1.10.1 - Automation Status tile border changed to solid green to match all other Cockpit cards (was still the old dashed placeholder style)
- v1.10.1 - Agent 6 (Admin Functions) was stuck in Draft/Inactive state in Dataverse, which made it show up oddly in the Power Automate panel - activated
- v1.10.2 - Release Notes now also has an "Agent (Flow) Changes" tab alongside "App Changes", listing each agent's current version and recent changes
- v1.10.2 - Replace button LED - the "processing" yellow blink never actually rendered because the upload and the flow call ran in the same formula with no repaint in between; the flow call now runs a moment later via a timer so the blink is visible
- v1.10.2 - All green status dots across the Cockpit (Files, Automation Status, System Health legend, Maintenance Domains) are now the same size and shape (14px circles) - previously a mix of text bullets and different-radius shapes
- v1.10.2 - Maintenance Domains - added a status dot in front of the count, matching the Files/Automation Status pattern
- v1.10.2 - Maintenance Domains - Edit now opens a quick "add new domain" form instead of duplicating the View link
- v1.10.2 - Header bar width aligned with the card row and Next Steps width for a clean edge-to-edge look
- v1.11.0 - Agent Monitoring page filled in - 6 tiles (one per agent) showing live status, last run result/duration, and status message straight from the DMP Command Agent Status SharePoint list
- v1.11.0 - Fixed System Health and Emails Processed tooltips/popups only showing one legend entry instead of all of them - the popup rows were missing an explicit setting that told them to keep their own height instead of sharing space with their siblings

### v1.9.3 - 2026-08-28

- Files and Maintenance Domains swapped positions (Files now top-right, Maintenance Domains bottom-left)
- Files - removed the Internal Domains row (redundant with Maintenance Domains), widened Name/Status/Last Updated columns to stop text truncation
- Maintenance Domains - compact new layout - count (green) - LOCATION - actions, e.g. "19 - INTERNAL - View Edit" and "6 - EXTERNAL - View Edit Replace"
- Operating State - "Operational Mode" caption shortened to "MODE"
- Change-notification LEDs (Critical/Warnings/Agents Active) - added a large invisible click area covering the whole tile including the LED itself - previously only the small number/label caught the click, making the LED very hard to tap
- v1.9.1 - added the missing second dash (between location and actions) to the Maintenance Domains format
- v1.9.2 - enlarged the invisible click area of the Replace button (Maintenance Domains - External) - was only slightly larger than the visible button, now has a generous margin on all sides without overlapping the neighboring Edit button or LED
- v1.9.3 - Release Notes text was getting cut off on longer entries - all entries now auto-size to their real content height
- v1.9.3 - Release Notes list is now scrollable, and a new "Copy to Clipboard" button copies the full version history as plain text
- v1.9.3 - Files card - right edge was poking out past its container on some screens - narrowed the card and tightened the Status/Last Updated column gaps further
- v1.9.3 - Heartbeat and Emails Processed ring cards - reduced the inner padding further to save horizontal space
- v1.9.3 - Emails Processed ring - the old plain-text tooltip could not show colours, so it was replaced with a click-to-open legend popup with real colour swatches matching each ring segment

### v1.8.1 - 2026-08-28

- Sidebar logo fixed - the source image had ~20-25% built-in padding on every side, so no resize ever actually made it bigger; it is now cropped to the real artwork and fills the sidebar width
- Major width reduction across the whole Cockpit header row to fit 100% browser zoom (was previously only fully visible at 75%) - narrower Operating State / Maintenance Domains / Files columns, narrower buttons, smaller ring padding, tighter gaps, narrower sidebar
- Operating State - "Mode" merged into the title area (color-coded like the screen border, blinks while switching), all labels made smaller/consistent, matching the Files card style
- Emails card - legend converted to a tooltip on the ring itself, so the ring can stay full size while the card gets much narrower
- Light mode - several too-light grey texts (Files status/time, Maintenance Domains counts) darkened for readability
- New diagnostics tile on the Admin page showing App.Width/App.Height for reporting layout issues
- v1.8.1 - fixed a sibling-indentation bug in this very file that blocked Studio from opening v1.8.0

### v1.7.0 - 2026-08-28

- Every sidebar menu item now has its own real screen instead of a "coming next" popup - Admin Functions moved to its own page, and Agent Monitoring / Operational Board / Audit Trail (Detail) / Configuration (Lists) / Maintenance each got a first "Coming Soon" placeholder screen (content to be filled in step by step)
- New change-notification LEDs on the Critical / Warnings / Agents Active header tiles - a small blue LED lights up (blinking) whenever one of these values changes since you last acknowledged it - click the tile to acknowledge; a new warning or critical lights the LED again automatically
- Version badge (top right) now links to this Release Notes page

### v1.6.4 - 2026-08-28

Frontend:
- Sidebar logo enlarged (280x188 -> 288x194 px), sidebar padding reduced
- Emails card (Agent 2 status) narrowed from 620 to 560px
- Next Steps - milestone text line-wrap fixed
- Display glitch during loading (garbled characters) fixed
- Reverted an over-aggressive container shrink that had truncated labels

Backend:
- New Error-ID system (e.g. [EC:A2-MBOXFOLDER-ND]) added across all 5 agents for faster troubleshooting
- Agent 1 & 2 performance improved (Excel counters now batched instead of updated per email)
- False "folder already exists" alarm fixed in Agent 3 & 4
- Copy-paste bug in Agent 1's error-mail move action fixed

### v1.6.1 - v1.6.3 - 2026-08-27/28

- Emails card visibility issue fixed (layout width)
- Several intermediate width-tuning iterations (superseded by v1.6.4)

### v1.5.x and earlier

- Older changes are documented in detail in the backlog document (DMP_COMMAND_Backlog.md), including the timestamp-format fix and earlier layout adjustments
- Starting with v1.6.4, this Release Notes page is updated with every new version

## Agent (Flow) Changes

### Agent 2 (E-Mail Inbox Treatment) - v1.0.7 (current)

- v1.0.7 (2026-09-01) - Internal sender classification now reads the "DMP Command Internal Domains" SharePoint list (Active = Yes) instead of the flat Internal_Domains.txt file - one fewer SharePoint call per e-mail
- v1.0.6 and earlier - audit counter writes batched (one combined read/write per run instead of per event), retry policies added to the critical Excel calls, error-ID codes ([EC:A2-...]) added for faster troubleshooting

### Agent 4 (Status Check) - v1.4.0 (current)

- v1.4.0 - now also reads the central Audit Trail table directly and returns the 20 most recent Critical (Failed) and Warning rows, used by the new Audit Trail (Detail) Cockpit page
- v1.3.0 (2026-09-01) - Internal Domains status check now reads the "DMP Command Internal Domains" SharePoint list (Active = Yes) instead of the flat Internal_Domains.txt file - same output fields (Exists/Count/LastModified) as before

### Agent 1 (Domains Extraction) - v1.0.7 (current)

- Audit counter writes batched (one combined read/write per run instead of per event), error-ID codes ([EC:A1-...]) added for faster troubleshooting
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added (feeds the Cockpit's Critical/Warning/Total-runs figures via Agent 4)
- 2026-08-13 - renumbered from "Agent 1" (unchanged number, but part of the system-wide sequential renumbering and Select+Join config-loading rebuild applied to all agents)

### Agent 3 (Emergency Report Management) - v1.1.4 (current)

- Handles Emergency Report uploads triggered from the Cockpit's Replace button, then regenerates External Domains
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added
- 2026-08-14 - alert-mail-then-move-to-folder error pattern added (matches Agents 1/2)
- 2026-08-13 - renamed from "Agent 3.01" as part of the system-wide sequential renumbering

### Agent 5 (Operational State Management) - v1.1.6 (current)

- Handles the Operating State toggle (Normal / DMP Operation) triggered from the Cockpit
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added
- 2026-08-14 - alert-mail-then-move-to-folder error pattern added (matches Agents 1/2/3)
- 2026-08-13 - renamed from "Agent 3.03 (YES File Management)" - the legacy Yes.txt file mechanism was fully decommissioned in favour of the CurrentOperationMode config value

### Agent 6 (Admin Functions) - v1.2.0 (current)

v1.5.x & older

