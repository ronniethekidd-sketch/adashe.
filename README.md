# Adashe

**Trustless rotating savings circles on Solana.**
Adashe (Hausa), esusu / ajo (Yoruba), chit funds, tandas, ROSCAs: hundreds of millions of people save this way, and today it runs on trust, WhatsApp groups and a treasurer who can disappear. Adashe moves the pot into a program with no admin key, so nobody can run off with it.

## How it works

1. Someone creates a circle: token (e.g. USDC), contribution, period, number of members (N).
2. Members join and lock **collateral = (N-1) x contribution**. When the circle fills, round 0 starts.
3. Each round, every member except that round's recipient pays one contribution.
4. When all N-1 have paid, the pot of **(N-1) x contribution** is released to the recipient (join order = payout order).
5. If someone misses the deadline, **anyone** can call `cover_default` and the defaulter's collateral pays their share. The pot is always whole.
6. After the last round, everyone withdraws remaining collateral.

Because the recipient never pays into their own round, each member owes exactly (N-1) contributions, and holds exactly (N-1) contributions as collateral. A member can never owe more than they have locked, so the pot is always fully funded.

## Program surface (7 instructions, ~600 lines, single file)

| Instruction | Who can call | Effect |
|---|---|---|
| `create_circle` | anyone | creates circle + vault PDA |
| `join_circle` | anyone, before join deadline | locks collateral, assigns slot |
| `contribute` | member, active round | pays contribution |
| `cover_default` | **anyone**, after round deadline | pays missing share from collateral |
| `release_payout` | **anyone**, when round funded | pays pot to the recipient's token account |
| `withdraw_collateral` | member, after completion | returns leftover collateral |
| `refund_unfilled` | member, if circle never filled | returns collateral, closes account |

No admin, no fee switch, no upgrade-controlled parameters inside the program. Cranks are permissionless so a stuck circle can always be unstuck.

## Build and test

```bash
yarn install
anchor keys sync        # replaces the placeholder program id in lib.rs + Anchor.toml
anchor build
yarn test               # bankrun tests (clock warping, no validator needed)
```

Toolchain: Anchor 0.30.1, Solana 1.18.x. Tests cover the happy path, default coverage, double-settle, wrong-recipient and wrong-token-account attacks, unfilled refund and parameter validation.

## Docs

- `docs/AUDIT_SCOPE.md`: what we want CertiK to review
- `docs/THREAT_MODEL.md`: invariants, attacks considered, known limitations
- `docs/MAINNET_PLAN_AND_ROADMAP.md`: path from audit to mainnet and beyond
- `SUBMISSION.md`: ready-to-paste Superteam Earn answers

## Status

Pre-audit prototype. Not deployed to mainnet. Do not use with real funds until audited.
