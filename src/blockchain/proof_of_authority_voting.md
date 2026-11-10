# A Blockchain-Based Tamper-Resistant Open Voting System for Local Elections


## How do I build a Proof of Authority blockchain where I can distribute authority to 100 people?

> Authorities can be civil societies, teachers picked up from a constituency block.


**Best choice: the Polkadot SDK (formerly Substrate).** It's the most mature Rust framework for building a custom chain, and PoA is a first-class setup.

**How to build it**
1. Start from the Polkadot SDK solochain template (`polkadot-sdk-solochain-template`).
2. **Block production:** Aura (round-robin among authorities). With 100 validators each gets a slot in turn.
3. **Finality:** GRANDPA, which gives deterministic finality and works fine at around 100 validators.
4. **Authority set:** replace the fixed genesis list with `pallet-session` plus `pallet-validator-set` (or `pallet-membership`) so validators can be added and removed at runtime.
5. **Distributing authority:** use `pallet-collective` (council or multisig-style voting), so adding or removing a validator needs, say, 67 of 100 votes. Don't leave `pallet-sudo` in production, since it gives one key total control.
6. Each validator generates session keys (Aura + GRANDPA), registers them on-chain, and runs a node.


## Run Full Node on Phone

> I want to run a full node on a phone. I expect 1 GB of data to last just 30 days until voting is complete, using Proof of Authority. Once the result is declared, one can stop the nodes.

**Yes, it's feasible for 30 days and ~1 GB.** Storage is no problem. The hard parts are uptime and connectivity.

**How to run it**
- **Android only.** Build the node for `aarch64-linux-android` (or compile inside Termux) and run it from Termux. iOS doesn't allow this.
- Keep the phone plugged in, disable battery optimization for Termux, and use a wake lock (`termux-wake-lock`) so the OS doesn't kill the process.
- Use `--state-pruning` and `--blocks-pruning` to keep storage small, though at 1 GB you may not need them.
- Phones sit behind carrier NAT, so they can't accept inbound connections. Run a few public bootnodes on cheap VPS machines and let phones dial out to them.

**The real risk: authorities on phones**
- Aura just skips a slot when an authority is offline, so block production continues.
- **GRANDPA finality needs more than 2/3 of authorities online.** If more than 33 of your 100 phones drop at once, finality stalls. For an election, that means results can't be declared as final.
- Phones drop often (calls, sleep, network changes, overheating).

**Safer setup**
- Phones run as **non-authority full nodes**. They sync and verify everything, so each person can check results independently.
- Authority nodes run on stable machines (cheap VPS or laptops). You can still spread authority across 100 people by letting each hold their own key on their own server.
- If every authority must be a phone, lower the finality threshold risk by using fewer, more reliable authorities, or have each person run a backup node with the same session keys (carefully, since two nodes signing at once can cause equivocation).

**For shutdown**
- Before stopping, export the final state and a signed result: `export-blocks` plus the final block hash and GRANDPA justification. That lets anyone verify the result later without the chain running.

## Dumbpipe Iroh Protocol for Substrate nodes behind NAT

[https://dumbpipe.dev](https://github.com/n0-computer/dumbpipe)

- **Give each node a fixed identity.** Start every Substrate node with a fixed `--node-key`, so its libp2p peer ID is known in advance. Dumbpipe separately gives each node an iroh endpoint ID or ticket.
- **Expose each node through dumbpipe.** On each node, run `dumbpipe listen-tcp --host 127.0.0.1:30333`. This publishes the node's P2P port under an iroh ticket, reachable by hole punching or relay fallback, with no public IP needed.
- **Publish the tickets as the discovery list.** Dumbpipe does not discover Substrate peers by itself. Put each node's ticket and peer ID in a shared list, such as the chain spec, the app config, or a signed Nostr event, so every node can find the others.
- **Connect to peers via local ports.** For each peer in the list, run `dumbpipe connect-tcp --addr 127.0.0.1:<local-port> <ticket>`. Then add `/ip4/127.0.0.1/tcp/<local-port>/p2p/<peer-id>` to the node's `--reserved-nodes`.
- **Lock the node to the listed peers.** Use `--reserved-only` and disable mDNS, so the node does not advertise or try to dial useless loopback addresses.
- **Keep the peer count small.** Give each phone about 5 to 10 tunnels, not all 1,000. GRANDPA and block gossip need a connected graph, not a full mesh.
- **Self-host an `iroh-relay`.** Hole punching fails on some carrier NATs, and traffic then goes through the relay. For a real election, run your own relay instead of the public defaults.
- **Embed iroh as a library on Android.** The CLI is awkward inside an app. A small Rust forwarder using the iroh crate, run in the same foreground service as the node, does the same job.

**Limit:** this is manual wiring that you maintain. Test it with real phones on mobile data before committing to it.

## Running it is as a App

**Yes on Android, no on iOS.**

**Android app approach**
- Compile the node for `aarch64-linux-android` with `cargo-ndk`.
- Package it as a native library (`.so` in `jniLibs`) rather than a bundled executable. Android 10+ blocks running binaries from app-writable storage, but allows running from the app's native library directory.
- Start the node from a **foreground service** with a persistent notification, so Android doesn't kill it.
- Request a wake lock and ask the user to exempt the app from battery optimization.
- Store the chain data in the app's private storage. At ~1 GB over 30 days, this is fine.
- Have the app handle the chain spec, bootnode addresses, and key setup, plus a simple UI showing sync status, block height, and the vote.
- Distribute it as a sideloaded APK to your 100 people. That avoids Play Store review, which may not like an embedded blockchain node.


## Few Nodes Getting Offline

> What if a few authority nodes go offline for a few hours? Can it still process transactions, right?


**Yes. A few authorities going offline for hours is fine.**

**Block production (Aura)**
- Authorities take turns by slot. If the authority for a slot is offline, that slot is simply empty and the next authority produces the next block.
- Transactions still get included, just with small delays. If 10 of 100 are offline, about 10% of slots are empty, so a transaction waits at most a few extra slot times.
- When the nodes come back, they sync and resume without any manual action.

**Finality (GRANDPA)**
- Finality keeps working as long as more than 2/3 of authorities are online. With 100 authorities, that means up to 33 can be offline.
- Up to that limit, finalized blocks keep advancing and nothing is lost.
- Beyond 33 offline, blocks are still produced and transactions still land in them, but nothing becomes finalized. Those blocks could theoretically be replaced by a different fork. When enough nodes return, GRANDPA finalizes everything in one catch-up step.

**Things to watch**
- **Stale or duplicate keys:** never run the same authority key on two nodes at once, which can cause equivocation. Only start a backup after the first is definitely off.
- **Clock drift:** Aura depends on accurate time, so make sure phones sync their clocks.
- **Long outages:** a few hours is fine. For days-long outages of a node, just let it resync from bootnodes.

**For your vote:** only treat a vote as counted once its block is **finalized**, not just included. Declare the result after the final voting block is finalized, then export that block's hash and justification.


## Multi-winner Star Voting

> Can I do large computations like multi-winner star voting for lakhs of users and produce a result in a Polkadot SDK PoA blockchain?

**Yes, but don't run the whole tally inside one block.** A runtime has a per-block weight limit (roughly 2 seconds of compute with 6-second blocks), and every node re-executes it. Collecting votes on-chain is easy. The tally itself needs a different approach.

**Size check**
- 3 lakh ballots × ~20 candidates × 1 byte ≈ 6 MB of data. Fine for 1 GB.
- Votes spread over 30 days is only about 10,000 per day. Throughput is not a problem.
- Multiwinner STAR is roughly voters × candidates × winners ≈ tens of millions of operations. That's trivial for a laptop but too heavy for one block.

**Three workable designs**

1. **Off-chain tally, on-chain agreement (simplest, recommended)**
   - Ballots are stored on-chain.
   - After voting closes and the last block is finalized, every authority runs the same deterministic tally program (written in Rust, with fixed tie-break rules) on the finalized ballots.
   - Each authority submits the result hash. The result is accepted when more than 2/3 match.
   - Anyone can rerun the program and verify it.

2. **Chunked on-chain tally**
   - Put the algorithm in a pallet and run it step by step over many blocks, for example 5,000 ballots per block, using `on_idle` or a "continue tally" extrinsic.
   - Fully on-chain and trustless, but slower to build.
   - Use integer or fixed-point math only (no floats), so every node gets identical results.

3. **Hybrid**
   - Keep running score sums on-chain as votes arrive. That's enough for single-winner STAR's first phase.
   - Run the multiwinner rounds with design 1 or 2, since they need all ballots.

**Things to decide early**
- **Privacy:** on-chain ballots are public. Secret ballots need encryption with threshold decryption or zero-knowledge proofs, which is much more work.
- **Eligibility:** one person, one vote needs a voter registry, for example a whitelist of keys or ID-verified accounts. Otherwise people can create many accounts.
- **Fairness of the algorithm:** freeze the exact STAR variant, tie-breaking and rounding rules before the election starts.


## More than 33% Nodes Offline

> What if more than 33% of the nodes go offline and come back online after some time?


**Short answer:** nothing is lost. Finality pauses while they're gone and catches up when they return.

**While more than 33% are offline**
- **Block production continues.** Aura keeps going, but slots belonging to offline authorities are empty. Say 40% are offline: the chain still produces blocks at ~60% of the normal rate, and transactions still get included.
- **Finality stalls.** GRANDPA needs more than 2/3 of authorities to vote, so the last finalized block stays frozen while the chain's head keeps growing. The gap between the head and the last finalized block widens.
- Blocks in that gap are **unfinalized**. They're probably fine, but they could in theory be reorganized, so don't treat votes in them as counted.

**When they return**
- Nodes sync the missing blocks from peers (or bootnodes) and resume.
- Once more than 2/3 are online and voting, GRANDPA finalizes the **entire backlog in one go**. It votes on the highest block it agrees on, and everything before it becomes final. No manual action is needed.
- There is no rollback of the unfinalized blocks as long as there was a single fork. The chain just continues.

**Things that can go wrong**
- **Long unfinalized gap:** nodes keep all unfinalized blocks in memory and state, so a very long stall (days) increases memory use and slows sync. For a stall of hours it's no problem.
- **Forks:** with fewer authorities online, a partitioned network could build competing forks. They resolve by longest-chain rule once connected, and only the winning fork gets finalized. Anything on the losing fork is dropped, including transactions, so users may need to resubmit.
- **Round restarts:** if GRANDPA gets stuck for a long time, a node restart or bad connectivity can slow its catch-up. Make sure nodes are on the same version and have enough peers.
- **Equivocation:** the danger when bringing nodes back is running a backup node with the same key while the original is still alive. Don't do that.

## PoA Blockchain Benefit over Nostr

> What the benefit of proof of authority chain over nostr?


**Q1. How do we know a vote was cast before the deadline?**
On Nostr, each relay stamps events with its own clock, and voters control their own `created_at`, so relays can disagree about votes near the deadline. That is why the document needs a week-long sync window. On a PoA chain, a vote is valid if it lands in a block before the closing block, and once that block is finalized the answer is permanent. Nobody has to reconcile clocks.

**Q2. What stops a single dishonest operator from dropping a vote?**
On Nostr, relays don't share a mempool, so a relay can say "I never received it" and nobody can prove otherwise. On a PoA chain, authorities gossip transactions to each other and take turns producing blocks. If one authority skips your vote, the next one can include it. Censorship would require every authority to refuse it, not one relay's admin.

**Q3. Who enforces one person, one vote and valid score ranges?**
On Nostr, every relay and every tallier must apply the rules themselves, and any difference in interpretation produces a different result. On a chain, the runtime checks eligibility, duplicate votes and score limits, and consensus rejects invalid votes. The rules live in one place and are the same for everyone.

**Q4. Can everyone agree on a single result without trusting a relay?**
On Nostr, each relay computes its own Merkle root, and you hope they match. On a PoA chain, there is one state root, and GRANDPA finality comes with a justification that anyone, even a light client on a phone, can verify. The result is one verifiable fact instead of several matching claims.

**Q5. What quorum do we need, and is it justified?**
The document mentions one-third, 51% and two-thirds for relay acknowledgment in different places, with no clear argument for any of them. A PoA chain with GRANDPA uses a fixed, well-studied threshold: more than 2/3 of authorities must agree for finality, and the system tolerates fewer than 1/3 faulty or offline ones.

**Q6. Can the history be altered or deleted later?**
On Nostr, relays can delete or withhold events, and kind 5 deletions are only requests, so the record's completeness depends on relay behavior. On a chain, each block commits to the previous one, so altering history is detectable by anyone holding a copy. For an audit trail, that tamper-evidence is the main point.

**Q7. Does the chain remove the "DID validators are gatekeepers" problem?**
No, and the document's argument here is partly misplaced. In your Nostr design, DID validators also decide who can vote. That is a property of the identity layer, not of the chain. The difference is that on a chain you can put changes to the voter registry behind a vote of the authorities, so changing the voter list requires a supermajority of the community instead of a few administrators.

**Q8. Where is Nostr still better?**
- **Reporting and evidence:** geotagged photos, comments, issue tracking and discussion fit Nostr well and don't need strict ordering.
- **Cost and simplicity:** no keys to guard for block production, no finality stalls, no runtime upgrades.
- **Phones and low connectivity:** Nostr clients are light, while authority nodes are heavy.

**Q9. So what should be built?**
Use both. Nostr carries the reports, evidence, discussion and audits. The PoA chain handles only the narrow job that needs strict ordering: registering voters, casting votes, closing the election, and recording the tally hash. That keeps the chain small, so the 1 GB and 30-day limits stay easy.
