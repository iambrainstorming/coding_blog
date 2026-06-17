# Web of Trust

Date: 17-06-2026
 

[Nostr Web of Trust](https://github.com/nostr-wot/nostr-wot)

Nostr has not natively implemented a Web of Trust (WoT) at the protocol level, as the protocol itself is an open system where anyone can publish and no one can be banned. Instead, WoT is provided by **third-party open-source infrastructure**, such as the **Nostr WoT** browser extension and oracle API, which allows users to build a local social graph based on their follow relationships.

Key developments in this user-controlled trust layer include:
*   **Conceptual Origin**: The concept derives from **PGP’s Web of Trust (1991)**, adapted for Nostr’s follow graph rather than cryptographic key signing.
*   **Infrastructure Launch**: The **Nostr WoT** project, which offers a browser extension for client-side queries and an oracle API for server-side lookups, was publicly announced and detailed in articles published in **February 2026**.
*   **Current State**: As of **June 2026**, this remains a decentralized, user-owned layer that Nostr apps can query to filter spam and prioritize content, rather than a built-in protocol feature.

* Your pubkey = center
* People you follow = 1 hop
* People they follow = 2 hops
* People beyond 3 hops = likely noise

is actually very close to how many practical Web-of-Trust systems work in Nostr and other decentralized networks. The idea is that trust decays with social distance.

## Why it works

Humans naturally trust through social connections:

* I trust Alice.
* Alice trusts Bob.
* Therefore Bob gets some trust from me.
* But not as much as Alice.

This is called **trust propagation** or **trust transitivity**. Researchers have studied it extensively and found that propagated trust can outperform purely global reputation systems in predicting trustworthy users. ([AAAI Publications][2])

In Nostr, Web of Trust is mainly used for:

* Spam filtering
* Content ranking
* Discovering new people
* Reducing bot influence

rather than establishing absolute truth. ([OpenSats][3])

## Why 1–3 hops is often enough

Most trust signal comes from nearby nodes.

### 1 hop

People you explicitly follow.

Highest confidence.

### 2 hops

Friends of friends.

Often still useful because they share social context.

### 3 hops

Weak signal but can help discovery.

### 4+ hops

Signal-to-noise ratio drops rapidly.

At this point you are reaching people with little meaningful connection to your network. Nostr WoT implementations often treat 3+ hops as increasingly untrusted. ([Nostr WoT][4])

This resembles the "three degrees of influence" idea observed in social network research: influence and trust tend to decay quickly with distance.

## Where Web of Trust fails

### 1. Echo chambers

If your network follows people with similar beliefs, trust propagation reinforces that bubble.

Example:

* Bitcoiners follow Bitcoiners.
* Rust developers follow Rust developers.
* Political groups follow their own side.

The system becomes good at finding insiders but bad at discovering outsiders.

### 2. Sybil attacks

An attacker can create:

* 10,000 fake accounts

but they still need trusted users to follow them.

This is why WoT is stronger than follower counts alone. Reputation is harder to fake than account creation. ([OpenSats][3])

### 3. Trust ≠ competence

Suppose:

* You trust a famous developer.
* They follow a friend.
* That friend starts posting medical advice.

The trust may propagate even though expertise does not.

This is a classic weakness of trust systems.

### 4. Community capture

Research on PGP's Web of Trust found strong community structures and hubs. Trust tends to cluster into communities rather than becoming globally reliable. 

## What research suggests works best

Pure hop-count is usually not enough.

Better systems combine:

1. Distance (1 hop > 2 hops > 3 hops)
2. Number of independent paths
3. Strength of relationships
4. Reputation of intermediaries
5. Time/history of interactions

For example:

* One path to Bob:

  * You → Alice → Bob

is weaker than

* Five independent paths:

  * You → Alice → Bob
  * You → Carol → Bob
  * You → Dave → Bob
  * You → Eve → Bob
  * You → Frank → Bob

Multiple independent paths are a strong signal. Modern Nostr WoT systems already incorporate path counts and graph structure rather than simple hop limits.
