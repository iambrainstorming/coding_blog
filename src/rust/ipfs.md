# Hard time with IPFS

Blockchain cannot store large files, and a mature IPFS is needed for true decentralization.

Accessing IPFS files with gateways hardly works; for example, if you upload a file to IPFS through one gateway, it is hardly accessible through other gateways. Furthermore, gateways can be censored.


## Monetize Iroh (IPFS) node

1. Store the blob hash and peer ID in the blockchain. If a user changes their peer ID, they need to update it in the blockchain. A user has to stake for each peer ID.
2. Host the Iroh [gateway](https://github.com/n0-computer/iroh-examples/blob/main/iroh-gateway/src/main.rs) and conduct random checks to find out if files of different hashes in the blockchain are accessible with the peer ID by the validators using the Iroh gateway.
3. The results of the daily random checks are stored in the blockchain and [governance through Schelling Coin is used](https://iambrainstorming.github.io/chapters/shivarthu/Shivarthu.html). The peer ID is incentivized or funds are deducted based on the random checks.

[Discussion](https://github.com/n0-computer/iroh/discussions/2356)

## DHT are slow

DHT are slow, blockchain can be used for storage of trackers, for easy peer discovery. Storing on blockchain, trackers can't be censored. One can use DHT along with direct peer discovery.

## Connecting Multiple Node Id and Fetching the Collection

The [gateway](https://github.com/n0-computer/iroh-examples/blob/main/iroh-gateway/src/main.rs#L358)  example connects a single node, and then fetching the file. 

What if I want to connect multiple node ids that have a file with the same hash, and download different parts of file from different node in parallel?

Node Ids are stored in blockchain so that it can't be censored. 

[Discussion](https://github.com/n0-computer/iroh/issues/2524)

## Wasm Iroh or libp2p

Wasm Iroh is needed to access files in the browser directly from the Iroh client, instead of fetching them through a gateway. Similar to [WebTorrent](https://webtorrent.io/)

Rust libp2p supports wasm. 


## Serveless hosting

Also wasm doesn't mean only for browser environment, Iroh need to also support edge serverless computing like wasmedge, [spin](https://github.com/fermyon/spin) or [everywhere computer](https://everywhere.computer/)

[Discussion](https://github.com/n0-computer/iroh/issues/1803)

Rust libp2p supports wasm

## Show the statistics

Fetching files can be slow at times; instead of leaving the user confused about why the file is failing to fetch, it needs to display statistics like seeders, leechers, and trackers.

## Split files into chunks

The following file sharing example, only connects to the single peer to fetch the complete file. What if I connect to multiple peers, and fetch part of file from different peers, just like bittorrent does. 

Doing it is not very hard, will try it while building dapps

[Discussion](https://github.com/libp2p/rust-libp2p/discussions/5508)
