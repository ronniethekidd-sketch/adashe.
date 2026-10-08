import * as anchor from "@coral-xyz/anchor";
import { Program, BN } from "@coral-xyz/anchor";
import { startAnchor, Clock, ProgramTestContext } from "solana-bankrun";
import { BankrunProvider } from "anchor-bankrun";
import {
  createMint,
  createAssociatedTokenAccount,
  mintTo,
  getAccount,
} from "@solana/spl-token";
import { Keypair, PublicKey, SystemProgram, LAMPORTS_PER_SOL } from "@solana/web3.js";
import { assert } from "chai";

const IDL = require("../target/idl/adashe.json");

const DECIMALS = 6;
const C = 100 * 10 ** DECIMALS; // 100 tokens per contribution
const PERIOD = 7 * 86_400; // weekly
const N = 3;
const POT = C * (N - 1);

describe("adashe", () => {
  let ctx: ProgramTestContext;
  let provider: BankrunProvider;
  let program: Program<any>;
  let payer: Keypair;
  let mint: PublicKey;

  type Actor = { kp: Keypair; ata: PublicKey };
  let actors: Actor[] = [];
  let nextId = 1;

  const pda = (seeds: Buffer[]) =>
    PublicKey.findProgramAddressSync(seeds, program.programId)[0];
  const circlePda = (creator: PublicKey, id: BN) =>
    pda([Buffer.from("circle"), creator.toBuffer(), id.toArrayLike(Buffer, "le", 8)]);
  const vaultPda = (circle: PublicKey) => pda([Buffer.from("vault"), circle.toBuffer()]);
  const memberPda = (circle: PublicKey, who: PublicKey) =>
    pda([Buffer.from("member"), circle.toBuffer(), who.toBuffer()]);

  async function warp(secs: number) {
    const c = await ctx.banksClient.getClock();
    ctx.setClock(
      new Clock(c.slot, c.epochStartTimestamp, c.epoch, c.leaderScheduleEpoch, c.unixTimestamp + BigInt(secs))
    );
  }

  async function newActor(): Promise<Actor> {
    const kp = Keypair.generate();
    ctx.setAccount(kp.publicKey, {
      lamports: 5 * LAMPORTS_PER_SOL,
      data: Buffer.alloc(0),
      owner: SystemProgram.programId,
      executable: false,
    });
    const ata = await createAssociatedTokenAccount(provider.connection, payer, mint, kp.publicKey);
    await mintTo(provider.connection, payer, mint, ata, payer, 10_000 * 10 ** DECIMALS);
    return { kp, ata };
  }

  async function bal(ata: PublicKey) {
    return Number((await getAccount(provider.connection, ata)).amount);
  }

  // Creates a circle with N members already joined (so it is Active).
  async function activeCircle() {
    const id = new BN(nextId++);
    const creator = actors[0];
    const circle = circlePda(creator.kp.publicKey, id);
    const vault = vaultPda(circle);
    await program.methods
      .createCircle(id, new BN(C), new BN(PERIOD), N, new BN(86_400))
      .accounts({
        creator: creator.kp.publicKey,
        mint,
        circle,
        vault,
        tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
        systemProgram: SystemProgram.programId,
      } as any)
      .signers([creator.kp])
      .rpc();
    for (let i = 0; i < N; i++) await join(circle, vault, actors[i]);
    return { circle, vault };
  }

  async function join(circle: PublicKey, vault: PublicKey, a: Actor) {
    await program.methods
      .joinCircle()
      .accounts({
        authority: a.kp.publicKey,
        circle,
        member: memberPda(circle, a.kp.publicKey),
        authorityToken: a.ata,
        vault,
        tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
        systemProgram: SystemProgram.programId,
      } as any)
      .signers([a.kp])
      .rpc();
  }

  const contribute = (circle: PublicKey, vault: PublicKey, a: Actor) =>
    program.methods
      .contribute()
      .accounts({
        authority: a.kp.publicKey,
        circle,
        member: memberPda(circle, a.kp.publicKey),
        authorityToken: a.ata,
        vault,
        tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
      } as any)
      .signers([a.kp])
      .rpc();

  const release = (circle: PublicKey, vault: PublicKey, recipient: Actor, tokenAcct?: PublicKey) =>
    program.methods
      .releasePayout()
      .accounts({
        circle,
        recipient: memberPda(circle, recipient.kp.publicKey),
        recipientToken: tokenAcct ?? recipient.ata,
        vault,
        tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
      } as any)
      .rpc();

  const cover = (circle: PublicKey, who: Actor) =>
    program.methods
      .coverDefault()
      .accounts({ circle, member: memberPda(circle, who.kp.publicKey) } as any)
      .rpc();

  async function expectFail(p: Promise<any>, code: string) {
    try {
      await p;
    } catch (e: any) {
      const s = JSON.stringify(e) + String(e);
      assert.include(s, code, `expected ${code}, got ${s.slice(0, 300)}`);
      return;
    }
    assert.fail(`expected failure ${code}`);
  }

  before(async () => {
    ctx = await startAnchor(".", [], []);
    provider = new BankrunProvider(ctx);
    anchor.setProvider(provider);
    program = new Program(IDL, provider);
    payer = (provider.wallet as anchor.Wallet).payer;
    mint = await createMint(provider.connection, payer, payer.publicKey, null, DECIMALS);
    for (let i = 0; i < 4; i++) actors.push(await newActor());
  });

  it("happy path: 3 members, everyone is paid once and keeps full collateral", async () => {
    const [a, b, c] = actors;
    const start = [await bal(a.ata), await bal(b.ata), await bal(c.ata)];
    const { circle, vault } = await activeCircle();
    assert.equal(await bal(vault), POT * N); // all collateral locked

    for (let round = 0; round < N; round++) {
      for (let i = 0; i < N; i++) if (i !== round) await contribute(circle, vault, actors[i]);
      await release(circle, vault, actors[round]);
    }
    const state = await program.account.circle.fetch(circle);
    assert.deepEqual(state.status, { completed: {} });

    for (let i = 0; i < N; i++) {
      await program.methods
        .withdrawCollateral()
        .accounts({
          authority: actors[i].kp.publicKey,
          circle,
          member: memberPda(circle, actors[i].kp.publicKey),
          authorityToken: actors[i].ata,
          vault,
          tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
        } as any)
        .signers([actors[i].kp])
        .rpc();
    }
    assert.equal(await bal(vault), 0);
    // Each member paid (N-1)*C and received (N-1)*C: net zero.
    for (let i = 0; i < N; i++) assert.equal(await bal(actors[i].ata), start[i]);
  });

  it("default: collateral covers the missing contribution and the pot is still paid", async () => {
    const [a, b, c] = actors;
    const { circle, vault } = await activeCircle();
    const before = await bal(b.ata);

    // Round 0: recipient a. b pays, c defaults.
    await contribute(circle, vault, b);
    await expectFail(release(circle, vault, a), "RoundIncomplete");
    await expectFail(cover(circle, c), "RoundNotExpired");
    await warp(PERIOD + 10);
    await cover(circle, c);
    await expectFail(cover(circle, c), "AlreadySettled"); // no double-cover
    const aBefore = await bal(a.ata);
    await release(circle, vault, a);
    assert.equal((await bal(a.ata)) - aBefore, POT);

    const cMember = await program.account.member.fetch(memberPda(circle, c.kp.publicKey));
    assert.equal(cMember.collateral.toNumber(), POT - C);
    assert.equal(cMember.defaults, 1);
    assert.equal(await bal(b.ata), before - C);
  });

  it("rejects: recipient paying own round, double contribution, wrong payout account", async () => {
    const [a, b, c, d] = actors;
    const { circle, vault } = await activeCircle();
    await expectFail(contribute(circle, vault, a), "RecipientDoesNotPay");
    await contribute(circle, vault, b);
    await expectFail(contribute(circle, vault, b), "AlreadySettled");
    await contribute(circle, vault, c);
    // attacker tries to redirect the pot to their own token account
    await expectFail(release(circle, vault, a, d.ata), "ConstraintTokenOwner");
    // wrong member passed as recipient
    await expectFail(release(circle, vault, b), "NotRecipient");
    await release(circle, vault, a); // correct call succeeds
  });

  it("refund: unfilled circle returns collateral after join window", async () => {
    const [a, b] = actors;
    const id = new BN(nextId++);
    const circle = circlePda(a.kp.publicKey, id);
    const vault = vaultPda(circle);
    await program.methods
      .createCircle(id, new BN(C), new BN(PERIOD), N, new BN(86_400))
      .accounts({
        creator: a.kp.publicKey,
        mint,
        circle,
        vault,
        tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
        systemProgram: SystemProgram.programId,
      } as any)
      .signers([a.kp])
      .rpc();
    const before = await bal(b.ata);
    await join(circle, vault, b);
    const refund = () =>
      program.methods
        .refundUnfilled()
        .accounts({
          authority: b.kp.publicKey,
          circle,
          member: memberPda(circle, b.kp.publicKey),
          authorityToken: b.ata,
          vault,
          tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
        } as any)
        .signers([b.kp])
        .rpc();
    await expectFail(refund(), "JoinWindowOpen");
    await warp(86_400 + 10);
    await expectFail(join(circle, vault, actors[2]), "JoinWindowClosed");
    await refund();
    assert.equal(await bal(b.ata), before);
    assert.equal(await bal(vault), 0);
  });

  it("rejects invalid circle parameters", async () => {
    const [a] = actors;
    const id = new BN(nextId++);
    const circle = circlePda(a.kp.publicKey, id);
    await expectFail(
      program.methods
        .createCircle(id, new BN(0), new BN(PERIOD), N, new BN(86_400))
        .accounts({
          creator: a.kp.publicKey,
          mint,
          circle,
          vault: vaultPda(circle),
          tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
          systemProgram: SystemProgram.programId,
        } as any)
        .signers([a.kp])
        .rpc(),
      "InvalidParams"
    );
  });
});
