# TESTING

```
COMPILE PASS ≠ RUNTIME PASS
SUBMITTED ≠ ACCEPTED ≠ FINALIZED ≠ EXECUTION SUCCESS ≠ POSTCONDITION PASS
```

## On-chain (GenLayer StudioNet, 2026-10-04)

Two wallets (author, other side), every term opened with the label `the other side`, in GenLayer Studio (Normal / Full
Consensus). Full table with inputs, hashes, results and read-back state: `RUNTIME_EVIDENCE.md`.

| Check | Cases | Result |
|---|---|---|
| **Check 1** — two sentences that both say `The amount is whatever …` | X4 → `FIXED`, Y4 → `TO_BE_AGREED` | PASS |
| **Check 2** — two sentences that both open `Each additional seat costs` | X3 → `FIXED`, Y3 → `TO_BE_AGREED` | PASS |
| **Check 3** — the same `record_amount(4000)` from the same wallet | X4 → FINAL (1 of 1), Y4 → PROPOSED (1 of 2) | PASS |
| counter-sign does not exist on a pinned term | reverted *This sum is fixed by the text; no counter-signature is needed* | PASS |
| reject does not exist on a pinned term | reverted *There is nothing to reject; the text fixes this sum* | PASS |
| a pinned sum is written once | second `record_amount` reverted *This sum is already final* | PASS |
| the proposer cannot counter-sign its own sum | reverted *The side that proposed the sum cannot counter-sign it* | PASS |
| reject reopens the term, the round count stays | AWAITING, amount 0, round_count 1 | PASS |
| both sides can propose | the other wallet proposed round 2 | PASS |
| two signatures from two wallets | FINAL 3500, countersigned_by = author, 2 of 2 | PASS |
| no counter-signature without a proposal | reverted *There is no proposed sum to counter-sign* (extra transaction) | PASS |

**Agreement.** Every open was accepted in its first round except X3, which needed two leader rotations before the
validators accepted `FIXED`. X3 is the case whose sum is reachable only through a rule ("the same as the first"). The
rubric was not changed after the run.

Test cases (5 pairs, same surface, opposite labels):

| Case | Text | Expected | On-chain |
|---|---|---|---|
| X4 | The amount is whatever the published list says on the day of order. | FIXED | FIXED |
| Y4 | The amount is whatever the parties arrive at in good faith. | TO_BE_AGREED | TO_BE_AGREED |
| X3 | Each additional seat costs the same as the first. | FIXED | FIXED |
| Y3 | Each additional seat costs whatever we later think reasonable. | TO_BE_AGREED | TO_BE_AGREED |
| X5 | The total is the unit price times the quantity ordered. | FIXED | not sent |
| Y5 | The total will be worked out together after the first month. | TO_BE_AGREED | not sent |
| X2 | The rate is set out in the schedule and does not move. | FIXED | not sent |
| Y2 | The rate is to be discussed once volumes are clear. | TO_BE_AGREED | not sent |
| X1 | The fee is 4,000 a month. | FIXED | not sent |
| Y1 | The fee will be settled between us nearer the time. | TO_BE_AGREED | not sent |

X4/Y4, X3/Y3, X5/Y5 and X2/Y2 are kill tests (the rubric does not hint at the mechanism); X1/Y1 is a definition check.

## Offline (before deploy)

- **genvm-linter** `lint`, `schema`, `typecheck`: pass; 9 methods (4 write, 5 view), no type errors.
- **Direct Mode tests** (`genlayer-test`, the real py-genlayer v0.2.16 SDK, model mocked): 56 passed — the threshold
  function for both certainties and the view matching it, the same 4000 giving FINAL and PROPOSED, both teeth, the
  proposer blocked from counter-signing and rejecting in every state, a third wallet refused by every write, the full
  propose → reject → propose → counter-sign loop, ten proposals and ten rejections leaving the term stuck in AWAITING,
  the ledger never rewritten, rounds scoped per term, all 25 revert strings each with a dedicated test, id formula and
  whitespace normalization, fail-safe, validator function, prompt fence, views. Sixteen deliberate faults (threshold
  fixed to 1, threshold inverted, self counter-sign allowed, self reject allowed, round count reset on reject, a pinned
  sum left PROPOSED, an open sum made FINAL at once, the other side recording on a pinned term, an open term limited to
  the author, no round cap, the fail-safe flipped, a stranger counter-signing, `countersigned_by` not stored, `FIXED`
  removed from the fence, signatures miscounted, the rejection note lost) were each caught by a test.
- **Rubric gate**: no word or word pair separates the two test classes; the rubric shares no content word with any of
  the 10 cases and avoids *settle, pinned, agree, agreed, whatever, total, rate, fee*.
- **Calldata**: every call used on-chain is between 97 and 161 bytes, under the 255-byte RPC limit.

## Consensus behaviour

The model is called once, in `open_term`. Validators re-run the classification and must agree on the exact label; a
disagreement rotates the leader or reverts the transaction (fail-closed) — X3 shows both rotations happening. Re-running
does not defend against prompt injection — the fence does. The three other writes are integer comparisons.

## What this run does NOT prove

- Four of the ten cases were sent, once each; label stability across repeated runs or validator sets is not measured.
  The validators were split on X3.
- The round cap (ten proposals) and amounts near 10^18 were tested offline only.
- Texts longer than about 163 characters (the contract allows 600) were not sent.
- Prompt-injection resistance rests on the fence and the reserved-token check; no adversarial model run was done.
