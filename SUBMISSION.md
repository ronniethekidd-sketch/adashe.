# Superteam Earn submission: CertiK Audit Credits

> Fill every [BRACKET] before submitting. Do not claim anything that is not true yet.

**Project name:** Adashe

**One-liner:** Trustless rotating savings circles (adashe / esusu / ajo) on Solana, with collateral-backed pots and no admin keys.

**Problem:** Rotating savings groups are one of the largest informal finance systems in the world, and across Africa most run on trust. Treasurers can vanish, members can take the pot and stop paying, and there is no recourse. People lose real savings.

**Solution:** A Solana program where each member locks collateral equal to their total obligation. The pot is paid out automatically when funded, and any missed contribution is covered from the defaulter's collateral by a permissionless crank. The program has no admin and no fee switch.

**Why it needs an audit before mainnet:** Every circle holds real stablecoins from people with little margin for loss. The core guarantee (the pot is always fully funded and the vault is always solvent) is exactly the kind of invariant an independent audit should verify.

**Codebase:** [GITHUB URL] (Anchor 0.30.1, ~600 LOC, 7 instructions, bankrun test suite)

**Audit scope:** See `docs/AUDIT_SCOPE.md` in the repo: seven specific questions on solvency, state machine, account validation, mint risk and clock use.

**Mainnet plan:** See `docs/MAINNET_PLAN_AND_ROADMAP.md`: audit, fixes, re-audit, Squads-multisig upgrade authority, mint allowlist and caps, closed pilot with real savings groups, public bug bounty, then caps lifted.

**Roadmap:** Guarded beta (Dec 2026), mobile-first product and naira on/off-ramps (Q1 2027), verifiable-random payout order and reputation-based collateral (2027).

**Team:** [YOUR NAME / Ronniethekidd], statistics student and founder based in Nigeria, [add other team members and relevant links].

**Demo / links:** [DEVNET PROGRAM ID], [DEMO VIDEO OR TWEET], [GITHUB URL]

**Honest status:** Pre-audit prototype. Devnet only. No mainnet funds at risk.
