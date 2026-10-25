# Tamarin models of MLS (RFC 9420): M0 and M1

Design notes for `m0.spthy` and `m1.spthy`. Section numbers refer to RFC 9420. Tags (D1, S5, E4, ...) are cited from comments inside the `.spthy` files and defined here. Results are whatever `make` records in `results/*.txt`; nothing is claimed here beyond those files. For the exists-trace lemmas, "verified" means Tamarin found the trace.

---

## M0: two-party create + Welcome (`m0.spthy`)

**Scope.** A creates a group (epoch 0), commits an Add of B (epoch 1), and B joins from the Welcome. No Update, no Remove, no ratchet tree, no PSKs, no external joins.

**Status.** Rules and lemmas written. Results are in `results/m0.txt`.

### Decisions (each takes the smallest option; alternative and revisit noted)

- **D1. The Add commit carries no path.** 12.4 allows this for an Add-only commit, and commit_secret is then the all-zero string. Consequence: epoch 1 gets no fresh entropy, so its secrecy rests on A's epoch-0 secret and on B's init_key, and 12.4 says such a commit gives no PCS for the committer. Alternative: populate the path and send B a path_secret in the Welcome. Revisit: M1, where Update and Remove force a path anyway.
- **D2. KDF.Extract(salt, ikm) is a free binary symbol `extract/2`, no equations.** Alternative: h(<salt, ikm>). Not taken because the transcript hashes are also h of a pair, and a dedicated symbol keeps the two apart for free. The all-zero string is the public constant 'zero'; the empty context of DeriveSecret is the public constant 'empty'.
- **D3. GroupContext = <gid, epoch, tree_hash, confirmed_transcript_hash>.** Dropped from 8.1: version, cipher_suite, extensions (one suite and no negotiation, so downgrade is out of scope). epoch is the constant '0' or '1'. tree_hash = h(tree), where tree is the tuple of leaves and leaf = <id, hpk(leaf_sk), pk(sig_sk)>. The transcript chain of 8.2 is kept in full, commit signature included: the confirmation tag means nothing without it. Revisit: epoch as a natural number in M1, real tree hash in M2.
- **D4. Trusted Authentication Service (5.3.1).** Each identity has exactly one long-term signature key, bound to it by a persistent fact that only the AS rule creates. A checks the KeyPackage's leaf against it; B checks the leaf of the GroupInfo signer against it. AS compromise (16.10) is out of scope.

### Other simplifications

- **S1.** GroupInfo travels in clear inside the Welcome. The RFC encrypts it under welcome_key (12.4.3.1), so welcome_secret is unused here and the adversary sees more than in the RFC. The HPKE context still binds the GroupSecrets to the GroupInfo. No claims about metadata confidentiality (16.4). Revisit: M3.
- **S2.** The Commit message is not sent. No member receives it in M0, and the adversary already knows everything in it except the commit signature, which no M0 rule verifies. Its content and signature still feed the transcript hash. Sending it roughly doubled Tamarin's case splits. Revisit: M1.
- **S3.** One signature per KeyPackage. The inner LeafNode signature (7.2), lifetime, capabilities and extensions are dropped.
- **S4.** GroupInfo.signer is implicit: always the first leaf (the creator).
- **S5.** B deletes the private init_key when it joins: a KeyPackage is consumed by Join, and RevInit is only allowed before that Join. No explicit requirement for this was found in the RFC; it is an assumption of this model and needs checking.
- **S6.** A moves to epoch 1 as soon as it commits, without waiting for the Delivery Service to echo the Commit (permitted by 3.2).
- **S7.** Joiner checks skipped: parent hashes (there are no parent nodes), cipher suite match, group_id uniqueness, lifetime.
- **S8.** A never adds itself: signature keys are unique among members (7.3) and each identity has one key (D4).
- **S9.** No epoch-1 state is stored. M0 ends at the Committed and Joined events; M1 adds the state that later commits continue from.

### State facts

| Fact | Meaning |
|---|---|
| `!Ltk($X, ~sig)` | X's long-term signature key (private). |
| `!Cred($X, pk(~sig))` | The AS binding; pk(~sig) is also public. |
| `KeyPkg($B, ~init, ~lk)` | Linear. B's private keys behind one published KeyPackage. Consumed when B joins (S5). |
| `!InitKey($B, ~init, kp)` | Read only by RevInit. |
| `Group0($A, ~gid, ~lk, ~es0)` | Linear. A's epoch-0 state. Consumed by the commit. |

### Key schedule (8, 11)

Epoch 0 (11): `~es0` is fresh; it is not derived from a joiner secret.

```
GC0      = <~gid, '0', h(leafA), 'empty'>
init0    = expand(~es0, 'init', 'empty')
ck0      = expand(~es0, 'confirm', 'empty')
tag0     = mac(ck0, 'empty')
interim0 = h(<'empty', tag0>)
```

Epoch 1:

```
tree1 = <leafA, leafB>
cth1  = h(<interim0, content, csig>)       content, csig: see Commit
GC1   = <~gid, '1', h(tree1), cth1>
js    = expand(extract(init0, 'zero'), 'joiner', GC1)      joiner_secret
es1   = expand(extract(js, 'zero'), 'epoch', GC1)          epoch_secret
ck1   = expand(es1, 'confirm', 'empty')
tag1  = mac(ck1, cth1)
```

### Messages on the wire

The network is the adversary; the DS is untrusted.

1. **KeyPackage, published by B (10).**
   ```
   leafB = <$B, hpk(~lk), pk(~sigB)>
   kp    = <hpk(~init), leafB>
   msg   = <kp, sign(<'KeyPackageTBS', kp>, ~sigB)>
   ```
   A accepts a kp from the network if the signature verifies under the key in its leaf and `!Cred` holds for the identity and key in that leaf.

2. **Commit, computed by A but not sent (S2) (12.4.1, 6.1).**
   ```
   content = <~gid, '0', $A, 'commit', <'add', kp>>          no path (D1)
   csig    = sign(<'FramedContentTBS', content, GC0>, ~sigA)
   ```

3. **Welcome, sent by A (12.4.3.1).**
   ```
   gi    = <GC1, tag1>
   gisig = sign(<'GroupInfoTBS', gi>, ~sigA)
   info  = <gi, gisig, tree1>                                in clear (S1)
   egs   = hseal(hpk(init), <'Welcome', info>, js)
   msg   = <h(<'KeyPackageRef', kp>), egs, info>
   ```
   B accepts if:
   - the reference matches its own kp, and egs opens under ~init;
   - gisig verifies under the signature key in the first leaf of tree1, and `!Cred` holds for that leaf's identity and key;
   - h(tree1) equals the tree_hash in GC1, and leafB is in tree1;
   - the tag recomputed from js and GC1 equals tag1.

### Adversary

Dolev-Yao on every message above, plus two reveals:

- `RevSig(X)`: X's long-term signature key.
- `RevInit(B, kp)`: the private init_key of a KeyPackage B has not yet used.

No reveal of group state (epoch or init secrets). That arrives with FS and PCS in M1.

### Lemmas

| Lemma | Statement |
|---|---|
| `exec` | The honest run exists, with no reveal. |
| `join_once` | A KeyPackage is used for at most one join. |
| `secrecy_committer` | A's epoch secret stays secret unless B's signature key leaked before the commit or the init_key of the KeyPackage A used leaked. |
| `agreement_state` | If B joins with signer A and A's signature key did not leak first, A committed that group, GroupContext and epoch secret (for some KeyPackage). |
| `secrecy_joiner` | B's epoch secret stays secret unless A's or B's signature key leaked before the join, or B's init_key leaked. |
| `agreement_keypackage` | As `agreement_state`, and A added exactly B's KeyPackage; needs B's signature key intact as well. |

Attack and control lemmas (exists-trace):

| Lemma | Statement |
|---|---|
| `attack_joiner_secrecy_needs_own_key` | B joins, the adversary knows the epoch secret, A's key never leaks, no init_key leaks. The adversary uses B's signature key to forge a KeyPackage for B with its own init_key, A adds it, and the adversary re-wraps the joiner secret to B's real init_key. |
| `attack_keypackage_agreement_needs_own_key` | The same forgery: B joins with a KeyPackage A never committed. |
| `control_forged_welcome` | With A's signature key leaked, B joins a group nobody committed. Confirms the reveal rules bite. |

---

## M1, slice 1: two members, one Update commit (`m1.spthy`)

**Scope.** Everything in M0, then B commits an update of its own leaf with a path and A processes it (epoch 2). Members now hold state and the adversary can reveal it. Decisions D1-D4 and simplifications S1-S9 of M0 carry over unchanged. Not yet: a third member, Remove, an Update proposal committed by someone else, A as committer, repeated commits.

**Status.** Results are in `results/m1.txt`.

### Decisions new in M1

- **E1.** State per member per epoch (9.2): init_secret, encryption_secret and the member's leaf private key. epoch_secret, joiner_secret and commit_secret are not kept; 9.2 requires them deleted once consumed. encryption_secret stands for all message keys of the epoch and is the "group key" in the lemmas (action `GroupKey`).
- **E2.** A member deletes an epoch's state when it moves to the next epoch (restriction `state_deleted_on_leaving`), so RevState is only possible while the member is in that epoch. RevState does not reveal the signature key (that is RevSig). A's epoch-0 state cannot be revealed.
- **E3.** Only B (leaf 1) commits and only A (leaf 0) processes. The mirrored pair of rules is not written, so results about "the committer" and "the other member" hold for this one assignment.
- **E4.** The update is an empty Commit with a path (12.4). B replaces its leaf key and sends one fresh path secret, sealed to A's leaf key with the provisional GroupContext as context (7.6). With two leaves this is the whole UpdatePath, except that the root key pair derived from the path secret is not kept: with two members nobody encrypts to it. commit_secret = expand(path_secret, 'path', 'empty').
- **E5.** The Commit is sent as content, signature and confirmation tag, with no membership_tag (6.2) and no PrivateMessage encryption. The adversary is stronger than in the RFC: forging a commit needs the sender's signature key and the confirmation tag, but no membership key.
- **E6.** The transcript chain continues (8.2): `interim1 = h(<cth1, tag1>)`, `cth2 = h(<interim1, content, csig>)`.
- **E7.** Epochs are the constants '0', '1', '2'. No rule can repeat, so no induction is needed. Revisit when commits can repeat.
- **E8.** B moves to epoch 2 as soon as it commits (as S6 in M0).

### State facts new in M1

| Fact | Meaning |
|---|---|
| `StA1($A, gid, GC1, interim1, init1, ~lk, leafA, leafB)` | Linear. What A needs to leave epoch 1. |
| `StB1($B, gid, GC1, interim1, init1, ~lk, leafB, leafA)` | Linear. What B needs to leave epoch 1. |
| `!Secrets($X, gid, ep, init, enc, lk)` | Read only by RevState (E1, E2). |

### Key schedule, epoch 2 (8)

```
init1 = expand(es1, 'init', 'empty')
enc1  = expand(es1, 'encryption', 'empty')
tree2 = <leafA, leafB2>          leafB2 = <$B, hpk(~lk2), pk(~sigB)>
prov  = <gid, '2', h(tree2), cth1>              provisional GroupContext
ct    = hseal(lkA, <'UpdatePathNode', prov>, ~ps)
content = <gid, '1', $B, 'commit', <'path', leafB2, ct>>
csig  = sign(<'FramedContentTBS', content, GC1>, ~sigB)
cth2  = h(<interim1, content, csig>)
GC2   = <gid, '2', h(tree2), cth2>
cs    = expand(~ps, 'path', 'empty')                      commit_secret
js2   = expand(extract(init1, cs), 'joiner', GC2)
es2   = expand(extract(js2, 'zero'), 'epoch', GC2)
tag2  = mac(expand(es2, 'confirm', 'empty'), cth2)
Commit on the wire: <content, csig, tag2>
```

A accepts if csig verifies under the key in B's current leaf, the new leaf key differs from the old one (12.4.2), ct opens under A's leaf key, and the tag recomputed from its own init1 equals tag2.

### Adversary

As M0 (Dolev-Yao, RevSig, RevInit), plus `RevState(X, gid, ep)`: the init_secret, encryption_secret and leaf private key that X holds for that epoch of that group.

### Lemmas new in M1

| Lemma | Statement |
|---|---|
| `exec_update` | The honest run reaches epoch 2 with both members agreeing, with no reveal. |
| `agreement_update` | If A processes a commit from B, B made that commit with the same GroupContext and epoch secret, unless A's or B's signature key leaked first. |
| `fs_epoch1` | Forward secrecy. The epoch-1 group key stays secret even if any state is revealed in epoch 2. Excuses: a state reveal in epoch 1, or any RevSig or RevInit. |
| `pcs_epoch2` | Post-compromise security. After B's commit the epoch-2 group key is secret even if B's own epoch-1 state had been revealed. Excuses: a state reveal in epoch 2, a reveal of the other member's epoch-1 state, or any RevSig or RevInit. |
| `control_state_reveal` | exists-trace. One epoch-1 state reveal is enough to learn the epoch-1 group key. |
| `attack_no_pcs_when_peer_compromised` | exists-trace. If A's epoch-1 state was revealed and B then commits, the adversary knows both secret inputs of epoch 2's key schedule (init_secret of epoch 1 and the path secret): only the committer heals. The lemma is stated on the inputs because that is cheap to prove. The same attack on the group key itself is at the end of `m1.spthy`, commented out: Tamarin finds the trace but needs about 10 minutes and 6 GB. |

The M0 lemmas are re-run unchanged inside `m1.spthy` as regression tests.

### Tool note

The rules are large enough that Tamarin's derivation checks time out at the default 5 seconds. Run with `--derivcheck-timeout=120`.
