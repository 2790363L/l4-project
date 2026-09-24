# Timelog

* Formal Verification of the Messaging Layer Security (MLS) Protocol
* Sam Lynch
* 2790363L
* Supervisor: Shahid Raza

## Instructions

* Log actual time spent, not planned time.
* Be specific enough that a stranger reading this could tell what you did (e.g. "installed
  Tamarin, worked through examples 1-3 in the manual" not "worked on project").
* Include reading, meetings, admin/email, and dead ends — not just coding/modelling time.
* Round to the nearest 15 minutes.

## Week 1

### 24/9/2026
* **09:30-13:00 3hrs30m:** Set up the project repo (plan.md mapped to the 24-week schedule, timelog, readme, .gitignore, folder structure), with the dissertation kept as a separate git repo synced only to Overleaf via its own remote. Filled in dissertation.tex from the UofG l4proj template (title, name, student ID) and rotated the Overleaf git token. Wrote a gameplan for the project (setup, reading, small Tamarin model, attack-hunting, SAPIC+ port) and pulled out methodology notes from the Basin et al. 5G paper to use as a template for the results chapter. Installed the Tamarin toolchain in WSL2 Ubuntu: Tamarin 1.12.0 from the release binary, then replaced the apt Maude 3.2 (unsupported) with Maude 3.5.1 via a symlink and removed the apt package, after debugging a mismatch between the old binary and the new prelude. Cloned the Tamarin repo for the examples and ran the interactive server on the Tutorial theory. Moved the repo from Windows Downloads into WSL, fixed line-ending noise (CRLF), set core.autocrlf, renamed the dissertation branch to main, and merged the Overleaf changes. Started reading Tutorial.spthy (PKI modelling, Fr/pub/fresh sorts, persistent vs linear facts) and worked through a Diffie-Hellman long-term key generation rule line by line.

### [DATE]
* **[TIME] [DURATION]:** [What you did]

## Week 2

### [DATE]
* **[TIME] [DURATION]:** [What you did]

<!--
Add a new "## Week N" section as you go. Example of a filled-in entry:

## Week 1

### Mon 22 Sept 2026
* **15:00-15:15, 0.25 hrs:** First supervisor meeting with Shahid Raza (Teams).
* **19:00-19:30, 0.5 hrs:** Wrote up meeting notes, drafted follow-up email.

### Tue 23 Sept 2026
* **10:00-12:00, 2 hrs:** Read Basin et al. CCS'18 5G authentication paper (sections 1-4).
-->
