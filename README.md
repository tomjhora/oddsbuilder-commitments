# OddsBuilder — public commitment log

This repository is an **append-only, public record of cryptographic commitments
to betting picks**, published *before* the games they refer to kick off.

Its only purpose is to let anyone check that a pick shown on
[oddsbuilder.app](https://oddsbuilder.app) after a game is the same pick that
existed before that game started — without taking OddsBuilder's word for it.

**How to check a pick: https://oddsbuilder.app/verify**

---

## Why this repository exists separately

The value of a commitment is entirely in its **timestamp**, and OddsBuilder's
own timestamp is the one thing a reader has least reason to trust.

A commit in a public GitHub repository carries a clock we do not control and a
history we cannot silently rewrite. A force-push is visible, and anyone who
cloned earlier holds the old history. That is the whole mechanism.

It is deliberately **separate from the product repository**, which is private.
A commitment published somewhere nobody can read is indistinguishable from no
commitment at all, while looking exactly like one.

---

## What is in a batch file

```
<season>/<seasonType>/<kind>/week-NN--<timestamp>.json
```

```json
{
  "canonVersion": "v1",
  "kind": "blueprint-ats",
  "season": 2026,
  "seasonType": "regular",
  "week": 1,
  "committedAt": "…",
  "earliestKickoff": "…",
  "pickCount": 12,
  "batchRoot": "…",
  "hashes": ["…", "…"]
}
```

**There are no game names, teams or pick sides in these files — only hashes.**

That is deliberate. Publishing which games we have picks on *before kickoff*
would give away part of the product. It costs a verifier nothing: you hash your
own revealed pick and look for that digest in the list. The files leak only the
**count**, which is public at reveal anyway.

`batchRoot` is the SHA-256 of the sorted hash list, so a batch file cannot have
entries added or removed without the root failing to re-derive.

---

## What a commitment proves, and what it does not

- **It proves:** the exact pick published after a game was already fixed before
  that game kicked off.
- **It does not prove the pick was good.** A losing pick committed honestly
  still hashes correctly. This is a check on record-keeping, not on a model.
- **It does not prove every game was committed.** It proves that what *was*
  committed was fixed in advance. Each batch states its own `pickCount`.
- **A batch committed after its first kickoff proves nothing.** The publishing
  tool refuses to write one, and `earliestKickoff` is published so you can
  check that yourself against the commit date.

---

## Canonical serialization

The exact byte-for-byte form that gets hashed — field list, ordering, null and
boolean encoding — is published and versioned at
**https://oddsbuilder.app/verify**.

The version is the **first line of the hashed text** (`ob-commit/v1`), so a
digest computed under one version can never be confused with another. A version
is never edited in place: if the format changes, the version increments and
older batches stay verifiable under the rules they were published with.

---

## If a pick does not verify

That is worth telling us about — and worth telling other people about. A
verification scheme nobody is willing to fail is not a verification scheme.
