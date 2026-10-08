# Mainnet Plan and Roadmap

Dates are targets, not promises, and assume audit credits are awarded.

## Phase 0: now (Oct 2026), audit-ready
- Program, tests, threat model and scope documented (this repo)
- Run full test suite on devnet; add property/fuzz tests (Trident) for the solvency invariant

## Phase 1: audit (Nov 2026)
- Submit scope to CertiK; answer auditor questions within 24h
- Fix all critical/high findings; re-audit the diff; publish the report

## Phase 2: guarded mainnet beta (Dec 2026)
- Deploy audited build. Upgrade authority held by a **Squads multisig** (3 of 5), moved to a timelock after beta
- Client-side **mint allowlist** (USDC first) and **caps**: small contribution ceiling and limited members per circle
- Real-time monitoring and alerts on vault/accounting anomalies
- Closed pilot with real savings groups (target: 5 to 10 circles of 5 to 12 people), recruited through local community networks in northern Nigeria
- Public bug bounty before lifting any caps

## Phase 3: product (Q1 2027)
- Mobile-first, WhatsApp-friendly UI: invite link, join, contribute, reminders
- Local on/off-ramp partnerships so members can fund and cash out in naira
- Verifiable-randomness payout ordering (optional per circle)

## Phase 4: scale (Q2 2027 onward)
- Reputation-based collateral reduction (post-audit, separately audited)
- Yield on idle vault balances through audited lending integrations (opt-in)
- Revoke upgrade authority once the program is stable and battle-tested

## Success metrics
Circles completed with zero loss of funds, default rate and recovery rate from collateral, number of first-time savers, time-to-first-circle for a new group.
