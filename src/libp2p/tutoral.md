# Building Blocks of libp2p

<iframe width="560" height="315" src="https://www.youtube.com/embed/Sbd7odDFT1w?si=rbC23G-GNeca9M7J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<https://libp2p.io/implementations/>

## Transports 

Enables libp2p move bytes from A to B

- Transports are core abstraction of libp2p
  - Enable connection establishment
  - Dialing and listening
- Current Transports:
  - TCP
  - QUIC
  - WebSockets
- Experimental:
  - WebRTC
  - Bluetooth

## Secure Channels

- Peer authentication and transport encryption
- Several security protocols supported
  - Noise
  - TLS 1.3

## Multiplexing

Multiplexing a connection is basically taking a connection and running a bunch of vitrual connections on top of that single connection. 

- Establishing a P2P connection may not be cheap or easy (e.g. hole punching, negotiation, handshake, etc.)
- Re-use established connections for serveral protocols
  - Applications can leverage already established connections.
- Several implementation of multiplexers available:
  - Language specific libraries for stream multiplex (Yamux, Mplex)
  - Transport protocol native mulitplexing capabilites (QUIC)

## NAT Traversal

Why overcome NATs and Firewalls? Most networks are not actually data centers well connected, but they are in home setting where they are behind a NAT or firewall. 

### Nov 22nd 2019, 63% of IPFS DHT we couldn't reach

Motivation: IPFS DHT crawl measurements (Nov 22nd 2019) showed that out of 4244 peers, 2754 were undialable (~63%)

Goal:

- Achieve global direct connectivity in hetergeneous networks
- No dependancy on central infrastructure

Added in 2021

- Transport Protocol: TCP, QUIC
- Relay Protocol (TURN-like): Circuit Relay V2
- Signaling Protocol: Direct Connection Upgrade through Relay (DCUtR)
- STUN like Protocol: AutoNAT

Next up: use this in WebRTC


## Peer Discovery

- Discover random peers (supporting certain services)
- Implementations
  - mDNS (Multicast DNS)
  - Rendezvous
  - GossipSub peer exchange

Routing - Kademlia DHT

- Distributed hash table
- Based on the Kademlia paper
- Operation:
  - FIND_NODE
  - GET_VALUE and PUT_VALUE
  - GET_PROVIDER and PUT_PROVIDER

Paper:
[Kademlia: A peer-to-peer Information System based on the XOR Metric](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf)


<iframe width="560" height="315" src="https://www.youtube.com/embed/1QdKhNpsj8M?si=LrYiw2pGoRq2HzfV" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



## Messaging - GossipSub

- Publish and subscribe
- Brokerless, self-regulating, no global knowledge
- Eager push and lazy pull

[GossipSub: Attack-Resilient Message Propagation in the Filecoin and ETH2.0 Networks](https://arxiv.org/abs/2007.02754)



## Data Exchange - Bitswap

- Message-oriented protocol
- Exchange blocks of data
  - Requests
    - WANT-HAVE
    - WANT-BLOCK
    - CANCEL
- Responses
  - HAVE
  - BLOCK
  - DONT_HAVE

[Accelerating Content Routing with Bitswap: A multi-path file transfer protocol in IPFS and Filecoin](https://research.protocol.ai/publications/accelerating-content-routing-with-bitswap-a-multi-path-file-transfer-protocol-in-ipfs-and-filecoin/delarocha2021.pdf)