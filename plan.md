# Plan

* Formal Verification of the Messaging Layer Security (MLS) Protocol
* Sam Lynch
* 2790363L
* Supervisor: Shahid Raza

Week-by-week plan for the whole project. Update this as you go along.

## Winter semester

* **Week 1** Install Tamarin/Maude (done). Work through the Tutorial theory. Write and prove a first toy model from scratch (key exchange with secrecy lemma plus an exists-trace lemma). Email Shahid Fri with status report and open questions (SAPIC+ scope, Tamarin paper reference, ProVerif MLS paper status, Sweden contact). Start the 5G paper.
* **Week 2** Finish the 5G paper (Basin et al., CCS'18) with examples/ccs18-5G open alongside. Read RFC 9420 (key schedule, TreeKEM, commit/welcome/add/remove/update). Second toy model with signatures and an agreement lemma, to practise authentication properties.
* **Week 3** Survey prior MLS attack work (Bhargavan et al., TreeSync, Cremers-Hale-Kohbrok, Alwen et al.). Write the security goals as precise lemma statements and define the reveal menu (leaf HPKE keys, signature keys, epoch/init secrets, PSKs). Start the fixed-group model skeleton (2-3 members, one tree shape, abstracted key schedule).
* **Week 4** Finish the fixed-group model with one commit epoch transition (n -> n+1). Prove basic secrecy lemmas.
* **Week 5** Authentication/agreement lemmas on the fixed-group model. Debug termination issues.
* **Week 6** Add `update`. Re-verify lemmas.
* **Week 7** Add `add` + `welcome`. Re-verify lemmas.
* **Week 8** Add `remove`, plus compromise rules for forward secrecy / post-compromise security.
* **Week 9** Consolidate the model and fix outstanding proof issues. First attack-hunting pass on external commits. Start the status report draft.
* **Week 10** Finish the status report draft. Attack-hunting: PSK injection.
* **Week 11 [PROJECT WEEK]** Sanity-check the status report with Shahid. Attack-hunting continues.
* **Week 12 [PROJECT WEEK]** Status report submitted.

## Winter break

* Buffer for anything that slipped. Draft the Background chapter (MLS, Tamarin, threat model), since it doesn't depend on results.

## Spring Semester

* **Week 13** Attack-hunting: membership rules (who can add members).
* **Week 14** Attack-hunting: welcome message handling.
* **Week 15** Attack-hunting: membership and epoch agreement edge cases.
* **Week 16** Consolidate findings, and classify each failed lemma as real finding vs. modelling bug.
* **Week 17** SAPIC+ port (if still in scope): translate the working Tamarin model, compare expressiveness and performance.
* **Week 18** Finish the SAPIC+ comparison, or (if dropped) extra depth on the Tamarin-only results.
* **Week 19** Dissertation writing: Introduction, Analysis/Design chapters.
* **Week 20** Dissertation writing: Implementation, Evaluation chapters.
* **Week 21** Conclusion, full draft complete. Send to Shahid for feedback.
* **Week 22** Revise from feedback. Fill any gaps with extra experiments if needed.
* **Week 23 [TERM ENDS]** Final proofreading, page-limit check, references.
* **Week 24** Dissertation submission deadline and presentations.
