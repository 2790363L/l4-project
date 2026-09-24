# Plan

* Formal Verification of the Messaging Layer Security (MLS) Protocol
* Sam [SURNAME]
* [STUDENT ID]
* Supervisor: Shahid Raza

Week-by-week plan for the whole project. Update this as you go along.

## Winter semester

* **Week 1** Confirm open questions with Shahid by email (SAPIC+ scope, exact Tamarin paper reference, ProVerif MLS paper status, Sweden contact intro). Install Tamarin and Maude; work through manual's toy examples.
* **Week 2** Read the 5G authentication paper (Basin et al., CCS'18) closely as methodology template — done. Start RFC 9420: key schedule, TreeKEM, commit/welcome/add/remove/update semantics.
* **Week 3** Finish RFC 9420 read-through. Survey prior MLS attack work: Bhargavan et al. (TreeKEM/F*), TreeSync F* paper, Cremers-Hale-Kohbrok (PCS healing), Alwen et al. (MLS security analyses). Confirm each is still current.
* **Week 4** Translate RFC 9420's informal security goals into precise Tamarin lemma statements (aliveness / weak / non-injective / injective agreement, and on what data). Define the reveal menu (leaf HPKE keys, signature keys, init/epoch secrets, PSKs).
* **Week 5** Design and build the fixed small-group Tamarin model: 2-3 members, one tree shape, one commit epoch transition (n -> n+1), abstracted key schedule.
* **Week 6** Prove basic secrecy and authentication lemmas on the fixed-group model. Debug termination issues.
* **Week 7** Extend model: add the `update` operation. Re-verify lemmas.
* **Week 8** Extend model: add `add` + `welcome`. Re-verify lemmas.
* **Week 9** Extend model: add `remove`, plus compromise rules for forward secrecy / post-compromise security.
* **Week 10** Consolidate model, fix any outstanding proof/termination issues. Start drafting status report.
* **Week 11 [PROJECT WEEK]** Finish status report draft. Sanity-check with Shahid.
* **Week 12 [PROJECT WEEK]** Status report submitted.

## Winter break

* Buffer for anything that slipped, plus start of attack-hunting reading (re-read 5G paper's minimal-assumption method with the working model in hand).

## Spring Semester

* **Week 13** Begin attack-hunting: strip assumptions one at a time from working lemmas, 5G-paper style. Start with external commits.
* **Week 14** Attack-hunting: PSK injection.
* **Week 15** Attack-hunting: membership rules (who can add members).
* **Week 16** Attack-hunting: welcome message handling.
* **Week 17** Attack-hunting: membership and epoch agreement edge cases.
* **Week 19** Consolidate attack-hunting findings; classify each failed lemma as real finding vs. modelling bug.
* **Week 20** SAPIC+ port (if still in scope): translate working Tamarin model, compare expressiveness/performance.
* **Week 21** Finish SAPIC+ comparison, or (if dropped) extra hardening/depth on the Tamarin-only results.
* **Week 22** Dissertation writing: Introduction, Background, Analysis/Design chapters.
* **Week 23 [TERM ENDS]** Dissertation writing: Implementation, Evaluation chapters.
* **Week 24** Dissertation writing: Conclusion, final proofreading, page-limit check. Dissertation submission deadline and presentations.
