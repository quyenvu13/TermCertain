# RUNTIME_EVIDENCE

Network: GenLayer StudioNet, chain 61999. Source SHA-256: `f97f02ce20d5395791eccdaabeabed3ba434155a2695e7bd0b9f8f7ccaf357a5`.
Every row: wallet, method, exact input, expected, tx hash, execution result, post-state from a view.
"EP" is the equivalence-principle output (the label the validators agreed on). A screenshot or a FINALIZED status alone is not evidence of execution.

Contract address: [`0x2719e3e965ced1c6bb227ae717A57287AE1D430a`](https://explorer-studio.genlayer.com/address/0x2719e3e965ced1c6bb227ae717A57287AE1D430a) · deploy tx `0xac267f4dac90e7ec472a46ce5845c45a255e156001ad31a7e6d044ae6545af62` (SUCCESS, FINALIZED)
Author wallet `0x923a09d0D6e5C242e36C3c1D2071835917cC0bDF` · other wallet `0x76DD809f34e0B72d9339bc509e1E19FaFEB445c2` · every term opened with label `the other side`.
Run date 2026-10-04, GenLayer Studio, Normal (Full Consensus). The table has 12 rows and **13 transactions** (row 12 is two); one more transaction outside the table is listed at the end.

## The pair that carries the concept: row 2 vs row 7

**The same method, the same wallet, the same number 4000 — two opposite states.** X4 and Y4 both read `The amount is whatever …`.

| | X4 (`signatures_required = 1`) | Y4 (`signatures_required = 2`) |
|---|---|---|
| `record_amount(…, 4000)` by the **author** `0x923a09d0D6e5C242e36C3c1D2071835917cC0bDF` | **FINAL** at once, `signatures_held = 1`, `binding = true` · row 2 `0x3c2bc030ea4147d176dc4c7f1667a4f1edf211a9267c24b0afaf1ff6c32f4423` | **PROPOSED**, `signatures_held = 1`, `binding = false` · row 7 `0xd9d3b63eb6a2cb8df5fa4fb3d774681cbea5f09e0e230159b5530f4138d27706` |

## Branch A — X4, `PINNED`

X4 = `The amount is whatever the published list says on the day of order.`

| # | Wallet | Method and input | Expected | Tx hash | Result | Status |
|---|---|---|---|---|---|---|
| 1 | author | `open_term(0x76DD…45c2, "the other side", X4)` | `FIXED`, `PINNED`, `signatures_required = 1` | `0xe038e991bb995afa36e0514ce0023d37ab51b83334dacbd4ded04116635d8ce5` | FINALIZED · SUCCESS · EP `FIXED`; `get_term`: PINNED, AWAITING, 0 of 1 | PASS |
| 2 | author | `record_amount(a7732854…5b80, 4000)` | FINAL at once | `0x3c2bc030ea4147d176dc4c7f1667a4f1edf211a9267c24b0afaf1ff6c32f4423` | FINALIZED · SUCCESS | PASS (read R1) |
| 3 | other | `countersign(a7732854…5b80)` | revert *This sum is fixed by the text; no counter-signature is needed* | `0xbec14f31dd23ffd9e4b4e686d3cc2d3ca24e2b428352a634c917f66943d6b26b` | ERROR · `[rollback]` that sentence (validators agree) | PASS (expected revert) |
| 4 | other | `reject_amount(a7732854…5b80, "Not agreed")` | revert *There is nothing to reject; the text fixes this sum* | `0x446953ce37d25b0a103bb31eef18ca2b0b18ecbce1b972f870c61790e33a905c` | ERROR · `[rollback]` that sentence (validators agree) | PASS (expected revert) |
| 5 | author | `record_amount(a7732854…5b80, 5000)` | revert *This sum is already final* | `0xf54744f771e75b5cb892a38fc1118a4f2d259c848569426ff190da349f45a9ae` | ERROR · `[rollback]` that sentence (validators agree) | PASS (expected revert) |

## Branch B — Y4, `OPEN_TERM`

Y4 = `The amount is whatever the parties arrive at in good faith.`

| # | Wallet | Method and input | Expected | Tx hash | Result | Status |
|---|---|---|---|---|---|---|
| 6 | author | `open_term(0x76DD…45c2, "the other side", Y4)` | `TO_BE_AGREED`, `OPEN_TERM`, `signatures_required = 2` | `0xcbbef9a160f0c586954c2571a21a3f2fa32137cc263bfd51dd538656dc55fb19` | FINALIZED · SUCCESS · EP `TO_BE_AGREED`; `get_term`: OPEN_TERM, AWAITING, 0 of 2 | PASS |
| 7 | author | `record_amount(ca831be4…2d3b, 4000)` | PROPOSED, 1 of 2, `round_count = 1` | `0xd9d3b63eb6a2cb8df5fa4fb3d774681cbea5f09e0e230159b5530f4138d27706` | FINALIZED · SUCCESS | PASS |
| 8 | **author** | `countersign(ca831be4…2d3b)` | revert *The side that proposed the sum cannot counter-sign it* | `0xec881bd71aa1286dc50add2a1989b40663acf838fe0c28dd45d71da5d8b57dc8` | ERROR · `[rollback]` that sentence (validators agree) | PASS (expected revert) |
| 9 | other | `reject_amount(ca831be4…2d3b, "Too high for the first month")` | AWAITING, `amount = 0`, `round_count` stays 1 | `0x43fcdee3bc9383a6d0f6cb43ecde396cb8efbfc756d74ce70b39ba10c7149326` | FINALIZED · SUCCESS | PASS (read R3) |
| 10 | **other** | `record_amount(ca831be4…2d3b, 3500)` | PROPOSED, `round_count = 2`, round 2 by the other wallet | `0x7380dd9a969d9ee88a535dfbee2cfec9735dff78b72ff1ab19d8135abc24ff3d` | FINALIZED · SUCCESS | PASS (read R3) |
| 11 | author | `countersign(ca831be4…2d3b)` | FINAL, `countersigned_by` = author, 2 of 2 | `0xe154ef66f9c5a4d048a32d7ff61bf08f4554fd18f1f1d1374235a80989705267` | FINALIZED · SUCCESS | PASS (read R2) |

## Branch C — X3 / Y3

| # | Wallet | Method and input | Expected | Tx hash | Result | Status |
|---|---|---|---|---|---|---|
| 12a | author | `open_term(…, "Each additional seat costs the same as the first.")` | `FIXED` | `0x29443158d6cb4ff15a7cbf1d7e1c0d4a0fbd0703fd540a6e9e5b21a6644b7603` | SUCCESS · EP `FIXED`, accepted after **two leader rotations** (see note) | PASS |
| 12b | author | `open_term(…, "Each additional seat costs whatever we later think reasonable.")` | `TO_BE_AGREED` | `0x5be8556b20fd7fe25d2a71eb69a6e73e14e6d1a380c98f3395b3247ef55b5a2b` | SUCCESS · EP `TO_BE_AGREED` | PASS |

**Note on X3.** The first two rounds did not reach agreement and the leader was rotated twice before the third round
accepted `FIXED`. X3 is the case whose sum is reachable only through a rule ("the same as the first"); validators were
split on it. Every other open was accepted in its first round. The rubric was not changed.

## Read-back (views, no transaction)

| # | View | Term | Returned |
|---|---|---|---|
| R1 | `get_term` | X4, after row 5 | FIXED / **PINNED**, **FINAL**, amount 4000, proposed_by author `0x923a…0bdf`, **countersigned_by ""**, round_count 1, **signatures 1 of 1, binding true** |
| R2 | `get_term` | Y4, after row 11 | TO_BE_AGREED / **OPEN_TERM**, **FINAL**, amount 3500, proposed_by other `0x76dd…45c2`, **countersigned_by author `0x923a…0bdf`**, round_count 2, **signatures 2 of 2, binding true** |
| R3 | `get_rounds(…, 0, 10)` | Y4 | round 1: 4000 by **author** `0x923a09d0d6e5c242e36c3c1d2071835917cc0bdf`, rejected *"Too high for the first month"* · round 2: 3500 by **other** `0x76dd809f34e0b72d9339bc509e1e19fafeb445c2`, no rejection |

Term ids (author `0x923a…0bdf`):
X4 `a7732854c638c966b2f2d8683279ad0efea720a710209d2f8d36ce49e91e5b80` ·
Y4 `ca831be4a1bae96306a5e94005190849235b30126ab155e3010137b9d4792d3b` ·
X3 `eeba34947a9253b4554f55fec7b1a4f04712222f6f362bd72ff2faa550026075` ·
Y3 `0a911c0141a52c2ee78b0490c15196019f0cfcf02ba32edef2f5f05659d256e4`

## Other transactions on this contract

| Tx | What happened |
|---|---|
| `0xd9e389d8d87f1570b76d8df434b3d440fc19ecafb73764638a06c9270983a0ce` | between rows 9 and 10, `countersign` on Y4 right after the rejection — reverted *There is no proposed sum to counter-sign* (the term was back in AWAITING); no state changed |
