# Threat Model

## Assets
Tokens in each circle vault: members' collateral plus the current round's contributions.

## Actors
Creator, members, permissionless crankers, malicious mint creators, anyone who can submit arbitrary accounts.

## Core invariants
1. **Vault solvency:** `vault_balance == sum(member.collateral) + contributions held this round`.
2. **Bounded liability:** each member owes (N-1) x contribution total (they never pay their own round) and locks exactly that as collateral. `cover_default` can never underflow collateral.
3. **Single settlement:** a member settles each round at most once (`settled == round` check), by paying or being covered, never both.
4. **Exact payout:** the recipient receives exactly (N-1) x contribution, only when `paid_count == N-1`, only to a token account owned by the recipient's authority.
5. **One recipient per round:** slots are unique (Member PDA keyed by circle + authority, slot = join counter).
6. **No admin:** nothing in the program lets any key move vault funds outside these rules.

## Attacks considered
| Attack | Mitigation |
|---|---|
| Redirect pot to attacker's token account | `recipient_token` must be owned by `recipient.authority` and match the circle mint |
| Pass wrong member as recipient | `recipient.slot == circle.round` constraint |
| Double contribution / contribute then get covered | `settled == round` guard on both paths |
| Recipient pays or is covered in own round | Explicit `RecipientDoesNotPay` check |
| Cover default before deadline | `now > round_deadline` check |
| Substitute a different vault | `address = circle.vault` pinned on every instruction |
| Same wallet joins twice to take multiple slots | Member PDA seeds make a second `init` fail |
| Overflow in pot / collateral math | Checked math; params validated at creation; release profile has overflow-checks on |
| Stuck circle (recipient never claims) | `release_payout` and `cover_default` are permissionless |
| Creator fills circle with sybils | Only locks the creator's own funds; every sybil must also post full collateral |

## Known limitations (documented honestly)
- **Payout order is join order.** Early slots get liquidity first. Fair-ordering via verifiable randomness is on the roadmap, not in v1.
- **Collateral is capital-heavy** (N-1 contributions). v1 trades capital efficiency for trustlessness. Reputation-based collateral reduction is roadmap, only after audit.
- **Mint trust.** The creator chooses the mint. A mint with a freeze authority (e.g. USDC) can freeze the vault. The client will allowlist mints; the program does not.
- **Token-2022 unsupported** (transfer hooks / fees would break accounting).
- **Late payment after deadline** is accepted until someone calls `cover_default`; this is intentional (no penalty beyond forfeiting the grace race).
- **Clock:** deadlines use `unix_timestamp`; minimum period of 1 hour makes small skew immaterial.
