# Freenet


## Is full data of contract is synced between peer?

Yes, but efficiently — not naively.Here's how it actually works:

**Only the delta (difference) is transmitted, not the full state.**

Naively we could transfer the entire state between peers, but this would be inefficient for larger states. Instead, Freenet transmits only the difference between states.

The sync process uses three contract-defined functions:

`summarize_state` returns a concise summary of the contract's state. `get_state_delta` compares the contract's state against the summary of another state and returns the difference — the "delta". `update_state` applies a delta to the contract's state, updating it to bring it in sync with the other peer's state.

**The step-by-step flow:**

PeerA sends a compact summary to PeerB. PeerB compares it against its own state — if they're identical, sync is complete. If different, PeerB calculates and sends only the delta back. PeerA applies the delta. Then the process repeats in reverse.

**Only subscribing peers hold the state at all.**

Freenet doesn't replicate every contract to every node. Only peers that subscribe to a contract store and sync its state. If no peers are subscribed, the data only exists locally and can disappear if that node goes offline.

**The contract itself defines the merge logic**, so what counts as "full state" is up to the app — River, for example, only considers the last 100 messages as the authoritative state, so that's all that ever gets synced.


## But when application launch first time full data has be fetched. Isn't it?
 
On first load, the full current state must be fetched.

When a peer subscribes to a contract for the first time, it has no local state at all — so there's nothing to summarize or diff against. It simply receives the complete current state from another peer that already has it.

After that initial full fetch, all subsequent syncs use the delta mechanism. So the flow is:

- **First time**: full state transferred
- **Every update after**: only deltas transferred

This is also directly relevant to why River's 100-message window matters practically — on first load, a new user fetches the entire current contract state, which includes those 100 messages, all member info, bans, configuration, etc. If the window were 10,000 messages, that initial fetch would be significantly heavier, especially on mobile or slow connections.

It's the same pattern used by Git, databases with replication, and most distributed systems — full snapshot on first sync, incremental diffs from then on.

## Freenet complements Iroh and rqbit

Freenet complements iroh and rqbit (BitTorrent) well. Freenet hosts indexing contracts that store iroh public keys or magnet links, enabling users to discover and directly connect to peers for fetching large data.


**Freenet's strength** is decentralized, replicated small state — contracts, indexes, metadata, membership lists. But it's not designed for large binary data transfer.

**iroh and rqbit's strength** is exactly the opposite — efficient peer-to-peer transfer of large blobs/files, but they need a way for peers to *discover* each other and find what's available.

So using Freenet as the **discovery/index layer** and iroh/BitTorrent as the **data transfer layer** plays to each system's strengths:

- Freenet contract = "where to find the data" (magnet links, iroh public keys, metadata) and reputation system
- iroh/rqbit = "actually fetching the data" (direct peer connections, chunked transfer)

## Direct connection api through Iroh

BitTorrent only transfers raw data, whereas iroh exposes a verifiable API backed by structured, cryptographically signed data — for example, a large SQLite database served directly over an authenticated peer connection.

- Freenet contract = decentralized index mapping identities/keys to iroh nodes
- iroh = authenticated API layer serving signed, structured data (e.g. from SQLite) directly over a direct peer connection
- rqbit/BitTorrent = still useful for large static blobs (videos, archives) where authorship matters less

## The Single Point of Failure Problem in Web3 Storage — and How Freenet + iroh Solves It

Blockchains cannot efficiently store even moderately large data — a 1MB or 10MB JSON file would incur prohibitive transaction costs. As a result, only a hash of the data is stored on-chain, and the actual data is delegated to an external storage layer.

A common approach is using an iroh node for this, but that introduces centralization risk. If that node goes offline or gets censored, the data becomes inaccessible. Falling back to another iroh node doesn't solve the problem — if that node refuses to serve the data or is malicious, it becomes a denial-of-service vector. Either way, it's a single point of failure that can take the entire application down.

This is where a CRDT-based approach like Freenet becomes valuable. Freenet keeps recent data — say, the last 10 days — replicated across peers in a decentralized manner. Multiple iroh nodes can independently fetch this cryptographically signed data from the Freenet network and use it to maintain their own local backup in a database like SQLite. Since the data is signed, any iroh node serving it can be verified for authenticity — a malicious node cannot tamper with it without detection. The result is a resilient architecture where no single node is a chokepoint, and the data remains available as long as any honest peer holds it.


## Tit for Tat Algorithm

Implementing a tit-for-tat algorithm within a Freenet contract would create a powerful incentive layer for iroh data availability. f you don’t serve their Iroh content, they won’t serve yours. This would create a mutually beneficial relationship, ensuring that neither party loses data during downtime.

This mutual dependency transforms data hosting from an altruistic act into a self-interested one — nodes are economically and practically motivated to stay online and honest, because their own data availability depends on it. Unlike token-based incentive systems used in networks like Filecoin, this requires no cryptocurrency or payment infrastructure — reciprocity itself is the currency.


## Freenet is in Alpha

It is worth noting that Freenet is currently in alpha, and network connectivity is not yet fully reliable — dropped connections and instability should be expected at this stage. Additionally, a mobile client is not yet available, which has meaningful implications for data availability. Mobile devices are inherently better suited for always-on participation in a peer-to-peer network than desktops or laptops, simply because mobile apps continue running in the background even when the screen is off. Laptops and desktops, by contrast, are frequently closed, shut down, or put to sleep — effectively removing that node from the network and reducing the number of peers holding a given contract's state.
