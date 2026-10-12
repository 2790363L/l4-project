# RFC evidence for M0 and M1

## M0

| ID | What the model relies on | RFC 9420 § | PDF p. | What the text says |
|---|---|---|---|---|
| D1 | An Add-only commit may omit the path | 12.4 | 83 | Path MUST be populated by default; MAY be omitted if the commit covers at least one proposal and none is path-required. |
| D1 | Without a path, commit_secret is the all-zero vector | 12.4.1 | 85 | Stated in 12.4.1, not 12.4 (the header cites 12.4). |
| D1 | A path-less commit gives no PCS for the committer | 12.4 | 83 | Called a "partial" commit: no PCS with regard to the committer. A commit with no proposals does provide it. |
| D3 | GroupContext fields | 8.1 | 59 | version, cipher_suite, group_id, epoch, tree_hash, confirmed_transcript_hash, extensions. The model drops version, cipher_suite, extensions. |
| D3, E6 | Transcript hash chain | 8.2 | 61 | confirmed[epoch] = Hash(interim[epoch-1] \|\| commit input); interim[epoch] = Hash(confirmed[epoch] \|\| confirmation_tag input). Both start empty at epoch 0. |
| (key schedule) | joiner_secret and epoch_secret derivation | 8 | 58 | Extract with commit_secret, then ExpandWithLabel "joiner" under GroupContext; then Extract with psk_secret (or 0), "epoch" under GroupContext. |
| (creation) | Epoch 0: confirmation_tag over the empty transcript hash, interim hash computed | 11 | 72-73 | As modelled. The fresh epoch_secret for epoch 0 is from the 2 Oct notes: not re-checked. |
| D4 | AS binds identity to signature_key | 5.3.1 | 29 | Credential validated when the AS verifies the identifiers are associated with the signature_key. |
| D4 | AS compromise out of scope | 16.10 | 107 | A compromised AS can assert any key/identity binding, allowing impersonation. |
| S1 | GroupInfo and part of the Welcome are not confidentiality-protected | 16.4 | 102 | Lists KeyPackage, GroupInfo and the unencrypted part of a Welcome as unprotected. |
| S1 | GroupInfo is encrypted under welcome_key inside the Welcome | 12.4.3.1 | 88-89 | Welcome struct has `encrypted_group_info` (p89, read). The welcome_key derivation was not re-checked. |
| S3 | KeyPackage / LeafNode fields | 7.2, 10 | 41, 70 | LeafNode: encryption_key, signature_key, credential, capabilities, leaf_node_source, lifetime (key_package case). |
| S4 | GroupInfo has a `signer` leaf index | 12.4.3.1 | 88 | Joiner verifies the GroupInfo signature with the key in the leaf at index signer. |
| S5 | Single use of KeyPackages; no requirement that the client delete the private init_key | 10, 16.8, 12.4.3.1 | 70, 106, 90 | §10: intended for single use, SHOULD NOT be reused except "last resort". §16.8: a used KeyPackage SHOULD be deleted from the publication system. §12.4.3.1 decrypts with init_key_priv and says nothing about deleting it. |
| S5 | init_key is intended to be deleted after the Welcome is processed | RFC 9750 5.1 | n/a | From a fetch summary. Read the section before quoting it. |
| S6 | A committer need not wait for its own Commit | 3.2 | 17 | The sender of a Commit does not necessarily have to wait to receive its own Commit back before advancing its state. |
| S7 | Joiner checks that M0 skips | 12.4.3.1, 7.3 | 88-91 | Only the GroupInfo signature check (p88) was re-read. |
| S8 | signature_key and encryption_key are unique among members | 7.3 | 44 | Listed as fields verified unique among the members of the group. |
| (Welcome) | GroupSecrets holds joiner_secret, optional path_secret, psks | 12.4.3.1 | 89 | As modelled; M0 uses joiner_secret only. |
| (attacks) | A leaked signature key lets an attacker create LeafNodes and KeyPackages as that member | 16.5 | 104 | Basis for the two attack lemmas and `control_forged_welcome`. |
| (reveals) | Old KeyPackage with a compromised private key leaves the new member unprotected | 16.6 | 105-106 | Basis for RevInit. |
| (lemmas) | Commit signature covers FramedContentTBS; confirmation_tag is a MAC | 6.1 | 34 | |

## M1 (additional)

| ID | What the model relies on | RFC 9420 § | PDF p. | What the text says |
|---|---|---|---|---|
| E1, E2 | Secrets are deleted as soon as consumed (init_secret, commit_secret, epoch_secret, encryption_secret) | 9.2 | 69 | This is where forward secrecy in the model comes from. |
| E2 | After processing an update, recipients delete used path/node secrets and replaced key pairs | 7.5 | 49 | MUST, stated at the end of 7.5. |
| E4 | An empty commit and Update/Remove/ExternalInit/GCE commits require a path | 12.4 | 83 | As modelled. |
| E4 | Path secrets are encrypted with the provisional GroupContext as context | 7.6 | 50 | EncryptWithLabel under node_public_key with the provisional GroupContext. |
| E4 | New leaf encryption_key must differ from the committer's current one | 12.4.2 | 86 | Checked by the receiver (the model's `Neq`). |
| E4 | Receiver recomputes the confirmation tag and compares | 12.4.2 | 87 | |
| E5 | membership_tag is a MAC for PublicMessage from members | 6.2 | 36 | The model omits it, so its adversary is stronger than the RFC's. |
| E6 | Transcript chain continues | 8.2 | 61 | See D3 above. |
| (lemmas) | FS between epochs comes from deleting old private keys | 16.6 | 105 | |
| (lemmas) | PCS takes effect once members process the commit | 12.4, 16.6 | n/a | From the 2 Oct notes. |
| (7.4) | Ratchet tree evolution | 7.4 | 44 | |
