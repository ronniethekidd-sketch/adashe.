# Audit Scope

## Request
Security audit of the `adashe` Solana program prior to mainnet launch, funded via CertiK audit credits.

## In scope
| Item | Detail |
|---|---|
| Repository | `https://github.com/ronniethekidd-sketch/adashe.` at commit `989357f890ddd3101a4639a3dae310d00e0fbea9` |
| Program | `programs/adashe/src/lib.rs` (single file, ~600 LOC Rust) |
| Framework | Anchor 0.30.1, `anchor-spl` token (classic SPL Token) |
| Instructions | create_circle, join_circle, contribute, cover_default, release_payout, withdraw_collateral, refund_unfilled |
| Accounts | Circle PDA, Member PDA, vault token-account PDA owned by the Circle PDA |

## Out of scope
- Frontend / mobile client (not yet built)
- Token-2022 mints (explicitly unsupported; classic SPL Token only)
- Upgrade-authority governance (covered separately in the mainnet plan)

## Questions we most want answered
1. **Solvency invariant.** Is `vault == sum(member.collateral) + current-round contributions` preserved in every path, including partial failures and every ordering of contribute / cover_default / release_payout?
2. **Collateral sufficiency.** Can any sequence of defaults make `cover_default` fail (InsufficientCollateral) and thereby freeze a round?
3. **Round state machine.** Can `paid_count`, `settled`, `round` or `status` be driven out of sync (e.g. member paying after being covered, recipient edge cases with N=2)?
4. **Permissionless cranks.** Any griefing or fund-redirection via `release_payout` / `cover_default` account substitution?
5. **PDA and signer handling.** Seed collisions, bump reuse, authority checks on token accounts, vault address pinning.
6. **Mint risk.** Freeze authority and malicious mints (creator chooses the mint): what should the program enforce vs. the client?
7. **Clock dependence.** Any exploitable behaviour from `unix_timestamp` skew at deadlines?

## Test evidence
`tests/adashe.ts` (bankrun): happy path with exact balance reconciliation, default coverage, double settle, recipient-pays-own-round, wrong payout token account, wrong recipient, unfilled refund, invalid params.
