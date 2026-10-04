TermCertain does not ask which side of a number is the wrong side, and it does not grade certainty on a scale. It asks whether a stranger holding this text and the relevant records could arrive at the sum alone — and it turns that answer into the number of signatures a later write needs before the sum binds anybody.

# TermCertain

GenLayer Intelligent Contract · StudioNet (chain 61999) · py-genlayer v0.2 (`# v0.2.16`)

**The contract holds no money and moves no money.** The texts talk about fees and prices; the contract only records a
number, how many signatures it needs and how many it has. It is not an escrow and it pays nobody.

| | |
|---|---|
| Contract | `contracts/TermCertain.py` |
| Address (StudioNet) | `0x2719e3e965ced1c6bb227ae717A57287AE1D430a` ([explorer](https://explorer-studio.genlayer.com/address/0x2719e3e965ced1c6bb227ae717A57287AE1D430a)) |
| Deploy tx | `0xac267f4dac90e7ec472a46ce5845c45a255e156001ad31a7e6d044ae6545af62` |
| Source SHA-256 | `f97f02ce20d5395791eccdaabeabed3ba434155a2695e7bd0b9f8f7ccaf357a5` |
| Tested on-chain | `RUNTIME_EVIDENCE.md` — 13 transactions in a 12-row table, one hash each; every row as expected |
| Testing summary | `TESTING.md` |

## What it does

The author records a text about a sum of money and names the other side. Validators read it **once**:

- `FIXED` — the text already pins the sum down, or gives a rule anyone can apply → `certainty = PINNED`;
- `TO_BE_AGREED` — the sum still waits on a fresh meeting of minds → `certainty = OPEN_TERM`.

The certainty sets the **signature threshold** of every later amount, through one function,
`_signatures_required` (1 for PINNED, 2 otherwise):

| | `PINNED` (1 of 1) | `OPEN_TERM` (2 of 2) |
|---|---|---|
| `record_amount` | the author only; **FINAL at once**, permanent | either side; **PROPOSED**, binds nobody yet |
| `countersign` | **reverts** *This sum is fixed by the text; no counter-signature is needed* | the other side, never the proposer → FINAL |
| `reject_amount` | **reverts** *There is nothing to reject; the text fixes this sum* | the other side, never the proposer → back to AWAITING |
| Loop | none — one write | propose → reject → propose…, up to 10 proposals |

`get_term` returns `signatures_required`, `signatures_held` and `binding` from the moment the term is opened, before
anybody records a number. `countersigned_by` stores the wallet that actually signed; on a PINNED term it is always empty.
Every proposal and rejection stays in a per-term ledger (`get_round`, `get_rounds`).

## Proven on StudioNet

| What | Transactions | Result |
|---|---|---|
| **The same sum, the same wallet, two states** | `record_amount(4000)` on X4 `0x3c2bc030…4423` · on Y4 `0xd9d3b63e…7706` | **FINAL (1 of 1)** / **PROPOSED (1 of 2)** |
| Two sentences that both say `The amount is whatever …` | X4 `0xe038e991…8ce5` · Y4 `0xcbbef9a1…fb19` | `FIXED` / `TO_BE_AGREED` |
| Two sentences that both open `Each additional seat costs` | X3 `0x29443158…7603` · Y3 `0x5be8556b…5a2b` | `FIXED` (after two leader rotations) / `TO_BE_AGREED` |
| Two methods that do not exist on a pinned term | countersign `0xbec14f31…b26b` · reject `0x446953ce…905c` | two different revert sentences |
| One write is one write | second `record_amount` on X4 `0xf54744f7…a9ae` | reverted *This sum is already final* |
| **The proposer cannot sign its own sum** | author countersigns its own 4000 `0xec881bd7…7dc8` | reverted *The side that proposed the sum cannot counter-sign it* |
| The loop, both directions | other rejects `0x43fcdee3…9326` · other proposes 3500 `0x7380dd9a…ff3d` · author signs `0xe154ef66…5267` | FINAL 3500, 2 of 2, two different wallets in the ledger |

## Methods

| Write | Who | Notes |
|---|---|---|
| `open_term(other_wallet, other_label, text)` | the author | the only model call |
| `record_amount(term_id_hex, value)` | PINNED: author · OPEN_TERM: either side | state → caller → 1 ≤ value ≤ 10^18 → round cap |
| `countersign(term_id_hex)` | OPEN_TERM, the side that did not propose | threshold → PROPOSED → not the proposer → a named side |
| `reject_amount(term_id_hex, note)` | OPEN_TERM, the side that did not propose | same order, then a 1–60 character note |

Views (JSON strings; an unknown id returns `"{}"`, never a revert): `get_term` (every field plus `outcome`,
`signatures_required`, `signatures_held`, `binding`), `get_round(term_id_hex, round_index)`,
`get_rounds(term_id_hex, offset, limit)`, `get_rubric`, `get_limits`. There is no preview or dry-run view.

Term id: `keccak256("TERM_CERTAIN:TERM:V1|" + author_lower + "|" + len(text) + "|" + text)`, where `text` has all
whitespace collapsed (Python `" ".join(text.split())`).

## Design choices

- **Fail-safe is `TO_BE_AGREED` — two signatures.** A wrong `FIXED` lets the author alone bind the other side to a
  sum of money it never agreed to, with no method to undo it. A wrong `TO_BE_AGREED` only makes the author wait for a
  signature. When it is unclear who agreed to what about money, the contract assumes nobody has.
- **The branches are not balanced in power, but they are balanced in price.** The author wants `FIXED`. To get it the
  text must contain enough for a stranger to compute the sum — and then the author is pinned to that same formula.
  `OPEN_TERM` keeps room to negotiate, at the cost of asking for a signature. The fail-safe runs against the author's
  wish, and the text is part of the id, so the same sentence cannot be reopened for another reading.
- **The threshold lives in one function.** `record_amount`, `countersign`, `reject_amount` and `get_term` only compare
  with `_signatures_required`; nothing else knows the number.
- **The proposer can neither counter-sign nor reject its own sum.** Without that check one wallet could record and sign,
  and two signatures would be one in disguise.
- **`round_count` never goes down.** The cap of 10 counts every proposal ever made, so the loop cannot run forever.
- **Prompt fence.** The model sees only the rubric, the other side's label and the text — no wallet, no state. `FIXED`
  and `TO_BE_AGREED` are reserved and rejected before the model call, so "the outcome is FIXED" cannot be smuggled in.

## How to try it in GenLayer Studio

Use **two wallets of your own**: author and other side.

1. Author: `open_term(<other>, "the other side", "The amount is whatever the published list says on the day of order.")`
   and the same with `"The amount is whatever the parties arrive at in good faith."`. Read `get_term`: 0 of 1 and 0 of 2.
2. Author: `record_amount(<each id>, 4000)` — FINAL on the first, PROPOSED on the second.
3. Other side: `countersign` the second → FINAL, 2 of 2. On the first it reverts.

One wallet alone can never take an `OPEN_TERM` term to FINAL — that is the mechanism, not a fault. Text and label may
not contain `FIXED` or `TO_BE_AGREED` in any letter case. Keep texts under about 163 characters so the calldata stays
under 255 bytes.

## Honest limitation

1. **The contract holds no money and moves no money.** It records a number, how many signatures it needs and how many
   it has. It is not an escrow.
2. **FINAL binds on this ledger, not in law.** The contract enforces nothing off chain, collects nothing and replaces no
   agreement. Its value is that the signature threshold is frozen before any number exists, so neither side can pick
   the threshold after seeing the number.
3. **A wrong `FIXED` is the main risk, and no method undoes it.** A sum would bind the other side without its
   signature. Nets: the fail-safe, `signatures_required` shown from the first transaction, and the error showing at the
   first `record_amount`, when the other side sees FINAL without having signed. On StudioNet the validators were split
   on X3 (a rule, "the same as the first") and needed two leader rotations to accept `FIXED`.
4. **A wrong `TO_BE_AGREED` leaves the author waiting.** The other side can stall, and a sum the text did pin stays
   unbound. The cost falls on the writer of the unclear sentence, and nobody is bound to a sum they did not accept —
   on purpose.
5. **The author declares the other side's wallet.** `open_term` refuses the author's own wallet and `countersign`
   refuses the proposer, but a second wallet is cheap — and here that matters most, because the whole value of the
   threshold of 2 is that the second signature comes from **someone else**. The contract cannot prove that two wallets
   are two people. This is the largest limitation.
6. **`MAX_ROUNDS = 10` and `MAX_AMOUNT = 10^18` are hard limits.** If the **tenth proposal is rejected**, the term stays
   in AWAITING for good: no new proposal, no FINAL, no way out. (Right after the tenth proposal the term is still
   PROPOSED and can be signed.) `MAX_AMOUNT` also means the contract is not meant for amounts in wei.

The contract accepts texts up to 600 characters, but `open_term` at the caps measures 759 bytes of calldata; only texts
up to about 163 characters fit the 255-byte limit, and longer ones would be refused by the RPC.

License: MIT.
