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

Further 2hr:

Worked through Tutorial.spthy: PKI modelling (Fr, pub/fresh sorts, ltk/pk, persistent vs linear facts with !), and went through a Diffie-Hellman long-term key generation rule line by line (let-bindings, action facts). Wrote my first Tamarin model from scratch-ish, src/toy_kex.spthy (Alice sends a fresh session key to Bob encrypted under his public key, plus PKI and key-reveal rules; wrote Bob_recv myself after working through Alice_send). Hit and fixed a parse issue (missing brackets/parentheses on the action fact) and a dangling variable (Bob's private key not bound on the left side), and a file-not-found from running it before the file existed. Ran it with tamarin-prover --prove: runs (exists-trace) verified, secrecy (holds unless Bob's key is revealed) verified, secrecy_no_excuse falsified. Read the falsified trace in the interactive UI (Register_pk, Alice_send, Reveal_ltk, attacker decrypt via d_0_adec) and worked out what each box does, and why Bob_recv doesn't appear in a minimal trace. Added an injection lemma (Bob accepts a key Alice never sent), which verified: the attacker encrypts his own key under Bob's public key, so public-key encryption alone gives confidentiality but no sender authentication. Next step is a signed variant to fix it.



### 25/9/2026
* **12:00-18:15 6hr15m:** Built src/toy_kex_signed.spthy from scratch: added signing builtins, wrote Alice_send (encrypt session key under Bob's public key, sign the ciphertext with Alice's long-term key) and Bob_recv (decrypt, verify signature via an Eq restriction). Debugged several issues along the way: construct-vs-read on the LHS (!Pk($B, pkB) not !Pk($B, pk(~ltk))), signing the ciphertext not the plaintext key, outputting the pair <ctx, sig> not the raw key, and collapsing double let-in blocks into one. Added agreement, runs, and secrecy lemmas. Agreement was falsified — read the attack trace in the GUI and identified a misdirection attack: the signature authenticated Alice as sender but didn't bind the intended recipient, so an attacker could relay Alice's signed ciphertext to a different Bob. Fixed by including the recipient's public key in the signed payload (sign(<ctx, pkB>, ltkA)) and verifying against it. All three lemmas now verified (agreement 12 steps, runs 8, secrecy 13). Spent a lot of time today just learning slowly how Tamarin works syntactically.

## Week 2

### 2/10/2026
* **[TIME] [DURATION]:** Started on M0, the first stage of the MLS Tamarin model, ahead of the planned 19 Oct start, to front-load modelling while proof runs are still short.

## Week 3

### 6/10/2026
* **[TIME] [DURATION]:** Continued the M0/M1 Tamarin modelling.

### 7/10/2026
* **[TIME] [DURATION]:** Meeting with Simon Bouget. Advised to drop post-compromise security from scope and focus on forward secrecy only, and to slow down and build the model from the bottom up rather than pushing ahead on the full thing.
