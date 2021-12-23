# Lightning Network

_Lightning Network and related off-chain protocols._

A few exceptional resources to get started:

[Master Lightning Book](https://github.com/lnbook/lnbook)

[Lightning Overview](http://dev.lightning.community/overview/)

[t-bast’s notes](https://github.com/t-bast/lightning-docs)

Lightning Network is a scaling solution to keep most transactions off-chain while leveraging the security of the bitcoin chain as an arbitration layer. There are several concepts to review before jumping into the domain. We will start small by covering lightning primitives, then apply these primitives to describe the Lightning Network.

Payments channels is a construct between two parties that commit funds and pay each other by updating a balance redeemable by either party. Moving funds between each part is near instant. Channels have a total capacity that is established by the on-chain funding transaction. Additionally, each party in the channel has their own balance. For example, a channel between Alice and Bob can have a 1 BTC capacity, but 30% of the bitcoin is owned by Bob. For Alice, this means that her local\_balance is 0.7 BTC while the remote\_balance \(Bob’s balance\) is 0.3 BTC.

To create the payment channel construction, a funding transaction is created on-chain. Any updates to the channel involves updating the commitment transaction.

Hash Time-Locked Contracts \(HTLCs\) allow transactions to be sent between parties who do not have a direct channels by routing it through multiple hops, so anyone connected to the Lightning Network is part of a single, interconnected global financial system.

Payment channels are the main workhorse of the Lightning Network. They allow multiple transactions to be aggregated into just a few on-chain transactions. In the vast majority of cases, someone only needs to broadcast the first and last transaction in the channel.

* The Funding Transaction creates the channel. During this stage, funds are sent into a multisig address controlled by both Alice and Bob, the counterparties to the channel. This address can be funded as a single-payer channel or by both Alice and Bob.
* The Closing Transaction closes the channel. When broadcast, the multisig address spends the funds back to Alice and Bob according to their agreed-upon channel amount.

channel updates

* In between the opening and closing transactions broadcast to the blockchain, Alice and Bob can create a near infinite number of intermediate closing transactions that gives different amounts to the two parties.
* For example, if the initial state of the channel credits both Alice and Bob with 5BTC out of the 10BTC total contained in the multisig address, Alice can make a 1BTC payment to Bob by updating the closing transaction to pay 4BTC/6BTC, where Alice is credited with 4BTC and Bob with 6BTC. Alice will give the signed transaction to Bob, which is equivalent to payment, because Bob can broadcast it at any time to claim his portion of the funds.
  * To prevent an attack where Alice voids her payment by broadcasting the initial state of 5BTC/5BTC, there needs to be a way to revoke prior closing transactions. Payment revocation roughly works like the following.
  * Alice must wait 3 days after broadcasting the closing transaction before she can redeem her funds. During this time, Bob is given a chance to reveal a secret that will allow him to sweep Alice’s funds immediately. Alice can thus revoke her claim to the money in some state by giving Bob the secret to the closing transaction. This allows Bob to take all of Alice’s money, but only if Alice attest to this old state by broadcasting the corresponding closing transaction to the blockchain.

Payment channels & revocable transactions

[great graphical overview](https://paychan.github.io/bitcoin-payment-channels-taxonomy/)

txn: Bob’s signature and a relative timelock \(Bob’s spend branch\); or Alice’s signature and a secret revocation hash provided by Bob \(Alice’s revocation branch\).

usually have multiple utxos. Once bob reveals his secret, alice can collect her spend TXO and rTXO.

revocable transaction script\_pub\_key: OP\_IF \# Bob’s spend branch - after the revocation timeout duration, Bob can spend with just his signature OP\_CHECKSEQUENCEVERIFY OP\_DROP OP\_ELSE \# Revocation branch - once the revocation pre-image is revealed, Alice can spend immediately with her signature OP\_HASH160 OP\_EQUALVERIFY OP\_DROP OP\_ENDIF OP\_CHECKSIG

recovcation keys used base points and blinding key. similar to bip32, keys derived using base key.

[Revocable Transactions](https://rusty.ozlabs.org/?p=450)

[HTLCs](https://rusty.ozlabs.org/?p=462)

[Enterprise Lightning Network](https://docs.google.com/presentation/d/1TyF0W3cZbkz4SyZG9qY7Is2pytC1GwSvs9KRKmYblFk/edit#slide=id.p)

### BOLTs <a id="78ad6867-3235-4f9c-ad2f-6009bc706fc6"></a>

[BOLT by BOLT](https://www.youtube.com/watch?v=Ysj2yobFMF4)

[Onion Routing with HTLCs](https://www.youtube.com/watch?v=toarjBSPFqI)

[Onion Messaging In Depth](https://rusty-lightning.medium.com/onion-messaging-in-depth-d8e384ee4184)

[Presentation by Rene](https://commons.wikimedia.org/wiki/File:Introduction_to_the_Lightning_Network_Protocol_and_the_Basics_of_Lightning_Technology_%28BOLT_aka_Lightning-rfc%29.pdf)

BOLT is the Basics of Lightning Technology.

The BOLT repo found [here](https://github.com/lightningnetwork/lightning-rfc) describes the specification for the Lightning Network.

1. BOLT \#0

   Provides a basic glossary defining terminology that is used throughout the rest of the specification.

2. BOLT \#1

   Describes the base message protocol including the TLV format and the setup messages.

   TLV is Type-Length-Value.

   Funny enough, the unicode code point for lightning is 0x2607. In decimal, 9735 which is also the default TCP port.

3. BOLT \#2

   Contains peer channel protocol lifecycle.

   A channel\_id is used to identify a channel. channel\_id = XOR\(funding\_txid, funding\_output\_index\)

   Before a channel is created, a temporary\_channel\_id is used which acts a nonce. This nonce is local and can be duplicate across the rest of the protocol.

   1. Channel Establishment
   2. Channel Close
   3. Normal Operation

      Once both nodes have exchanged funding\_locked, the channel is used to make payments with HTLCs.

4. BOLT \#3

   Describes transaction and script formats.

5. BOLT \#4
6. BOLT \#5

   Channels can end with a mutual close, unilateral close, or a revoked transaction close.

   In a mutual close, local and remote nodes agree to close. They generate a closing transaction.

   In a unilateral close, one side publishes its latest commitment transaction.

   In a revoked transaction close, one party is cheating and publishes an oudated commitment transaction.

   A commitment transaction has up to six types of outputs:

   1. local node’s main output: Zero or one output, to pay to the local node’s delayed\_pubkey.
   2. remote node’s main output: Zero or one output, to pay to the remote node’s delayed\_pubkey.
   3. local node’s anchor output: one output paying to the local node’s funding\_pubkey.
   4. remote node’s anchor output: one output paying to the remote node’s funding\_pubkey.
   5. local node’s offered HTLCs: Zero or more pending payments \(HTLCs\), to pay the remote node in return for a payment preimage.
   6. remote node’s offered HTLCs: Zero or more pending payments \(HTLCs\), to pay the local node in return for a payment preimage.

   If the local node publishes its commitment transaction, it will have to wait to claim its own funds, whereas the remote node will have immediate access to its own funds.

7. BOLT \#7

   P2P

8. BOLT \#8
9. BOLT \#9
10. BOLT \#10
11. BOLT \#11

    Invoice spec.

12. WIP: BOLT \#12

    BOLT 12 describes a new invoice format and flow called Offers.

    The Draft of the PR can be found [here](https://github.com/lightningnetwork/lightning-rfc/pull/798).

    The flow described is the following:

    1. Receiver publishes an offer.
    2. Payer requests a new unique invoice over LN using the offer.
    3. Receiver responds with a unique invoice.
    4. Payer pays the invoice.

    There are a number of improvements over BOLT11.

    Payment proof is designed to allow the payer to prove that they were the unique payer.

    Merkle tree is used to be able to prove only specific fields of the invoice, not the enture invoice!

    Some offers are periodic, meaning that payments are expected on a recurring period. This allows for new applications that require subscription-based payments.

### Implementations <a id="d391f30a-f29e-4e7d-a6be-52fa459050dc"></a>

There are several implementations following the BOLT specification.

1. LND

   [Exploring LND 0.4](http://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2018-04-20-laolu-osuntokun-exploring-lnd0.4/) [LND 0.6-Beta](http://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2019-05-02-conner-fromknecht-lnd-0.6-beta/)

### anecdotal example <a id="2ca5b469-a85e-40d4-9fa1-8c43938ec837"></a>

Suppose Alice has a channel with Bob, who has a channel with Carol, who has a channel with Dave: A&lt;-&gt;B&lt;-&gt;C&lt;-&gt;D. How can Alice pay Dave? Alice first notifies Dave that she wants to send him some money. In order for Dave to accept this payment, he must generate a random number R. He keeps R secret, but hashes it and gives the hash H to Alice.

Alice tells Bob: “I will pay you if you can produce the preimage of H within 3 days.” In particular, she signs a transaction where for the first three days after it is broadcast, only Bob can redeem it with knowledge of R, and afterwards it is redeemable only by Alice. This transaction is called a Hash Time-Locked Contract \(HTLC\) and allows Alice to make a conditional promise to Bob while ensuring that her funds will not be accidentally burned if Bob never learns what R is. She gives this signed transaction to Bob, but neither of them broadcast it, because they are expecting to clear it out later. Bob, knowing that he can pull funds from Alice if he knows R, now has no issue telling Carol: “I will pay you if you can produce the preimage of H within 2 days.” Carol does the same, making an HTLC that will pay Dave if Dave can produce R within 1 day. However, Dave does in fact know R. Because Dave is able to pull the desired amount from Carol, Dave can consider the payment from Alice completed. Now, he has no problem telling R to Carol and Bob so that they are able to collect their funds as well.

Alice knows that Bob can pull funds from her since he has R, so she tells Bob: “I’ll pay you, regardless of R, and in doing so we’ll terminate the HTLC so we can forget about R.” Bob does the same with Carol, and Carol with Dave.

Now, what if Dave is uncooperative and refuses to give R to Bob and Carol? Note that Dave must broadcast the transaction from Carol within 1 day, and in doing so must reveal R in order to redeem the funds. Bob and Carol can simply look at the blockchain to determine what R is and settle off-chain as well.

### Lightning Conf 2019 Berlin <a id="fb08e8d6-8c7a-4a98-8c64-bc01071d5d9a"></a>

[electrum slides on lightning](https://www.electrum.org/talks/lightning/presentation.html#slide1) Circular routes: send to self. Suggestions

* do not accept random peers
* disallow invoices to blacklisted pubkeys

Command line tools

* LNDmanage by @bitromortac
* Balance of Satoshis by @alexbosworth
* Rebalance-LND by @C-Otto

Make Me an Offer \(Bolt 12\) introduced.

LSAT

* Macaroon - cryptographic bearer credential
* Delegation possible
* Chained HMAC construction
  * Secret root used to derive all others
* Fine grained permission

Hedging the Chain

* Bitcoin fee market
* “Every biz using the blockchain is inherently short blockchain fees”
* Derivatives traditionally used as a hedge
* Corn farmers inherently long corn
* They short corn futures as a hedge

Liquidity

* No pairwise trades
* different sources of liquidity is not the same
* Set outbound liquidity to the same fee
* Varied inbound liquidity
* Make liquidity a pairwise market
* External settlement mechanisms
* Circular rebalancing

Attacks

* Set min chan size …too many channels causes performance issues
* Create a bunch of hold invoices and drain balance
* Stealing free fees, someone sets up intermediate node between invoice and collects fees.

[htlcs are harmful](http://diyhpl.us/wiki/transcripts/stanford-blockchain-conference/2019/htlcs-considered-harmful/)

### Discrete Log Contracts <a id="934a5090-e785-43ad-a057-904335285bc4"></a>

[intro](https://medium.com/@gertjaap/discreet-log-contracts-invisible-smart-contracts-on-the-bitcoin-blockchain-cc8afbdbf0db)

### Security <a id="edd3ad2e-56c3-400c-9b3b-7037ec46e980"></a>

[securing lightning nodes by devrandom](https://medium.com/@devrandom/securing-lightning-nodes-39410747734b)[link to the lightning-signer project on GitLab](https://gitlab.com/lightning-signer)[key mgmt](https://suredbits.com/lightning-101-for-exchanges-security-part-3-private-key-management/)

[blackmail attack](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-June/002735.html)

1. LSAT

   Lightning Service Authentication Token[lsat talk](https://docs.google.com/presentation/d/1QSm8tQs35-ZGf7a7a2pvFlSduH3mzvMgQaf-06Jjaow)

   using macaroon based bearer API credential with lightning network payment

2. Key management

   _Overview of each key in the channel lifecycle._

   [talk on key mgmt](https://docs.google.com/presentation/d/1_-FF0U2AXuhBxEzW9J_IrYxvRi1SS2MYwJl0QeIcqbI) need onchain hot wallet to open channels \(only need once\)

   1 of 2 keys must be hot for the funding transaction. If counterparty gets key, funds are lossed. If 3rd party gets it, they must collude.

   Commitment secret: must be hot. Used to generate “local\_pubkey” and “remote\_pubkey” Used to derive subsequent secrets and public keys. If leaked, peer can steal all money in commitment txn.

   Revocation basepoint secret: can be cold. Used to claim peer funds if they try to cheat. Can be cold if accessible before “to\_self\_delay”

   If your counterparty gets access to this key, they can claim their funds in their to\_local output immediately by circumventing the locktime

   Payment basepoint secret: claim money from the “to\_remote” output on peer commitment txn. can be cold if peer gets access to this key, all funds can be taken in the “to\_remote”.

   Delayed Payment Basepoint Secret: claim money on “to\_local” output of commitment txn. can be cold.

   HTLC Basepoint Secret: secret needed to sign for HTLCs. must be hot.

   hosted channels[gist on hosted channels](https://gist.github.com/btcontract/d4122a79911eef2620f16b3dfe2850a8) interesting idea but need to look more into security assumptions..

### Privacy <a id="9ca19ba3-849a-4949-8cf5-404b6ad8ed94"></a>

[An Empirical Analysis of Privacy in the Lightning Network](https://arxiv.org/abs/2003.12470)

### Routing <a id="cf9563b0-5163-45d6-96dd-460dcdb6aa9b"></a>

Routing involves routing a lightning payment through either a public or private channel.

Routing is generally constructed for a specified payment amount. Other considerations, however, includes value of open channels, decision to make new channels, re-balancing decisions, multi-path payments or multi-part payments \(MMP, formerly AMP\).

[amount independent routing](https://medium.com/coinmonks/amount-independent-payment-routing-in-lightning-networks-6409201ff5ed)

One of problem in routing is payment privacy. Two proposals to increase the privacy of paymnet senders and recipients are rendevous routing and route blinding.

1. Rendezvous Routing

   Rendevous routing is a [proposal](https://lists.linuxfoundation.org/pipermail/lightning-dev/2018-November/001498.html) aimed to protect the privacy of payments on the lightning network. In the initial proposal, an argument is made that private channels should not be revealed to payers. The solution is to have the payee choose one or more routes from certain third-party nodes on the public network to himself, and pass sphinx-encryped blogs for those routes to the payer. Then, the payer complets the route by finding routes from himself to the selected third-party nodes.

   [Rendezvous mechanism on top of sphinx](https://github.com/lightningnetwork/lightning-rfc/wiki/Rendez-vous-mechanism-on-top-of-Sphinx)

2. Route Blinding

   Route Blinding is currently a [proposal](https://github.com/lightningnetwork/lightning-rfc/blob/route-blinding/proposals/route-blinding.md) that aims to provide recipient anonymity by blinding an arbitrary amount of hops at the end of an onion path. Like rendezvous routing, this proposal is aimed and hiding the final portion of the route from the sender. The recipient chooses an “introduction point” and a route to himself from that point. The recipient blinds each node and channel for that route with ECDH. This blinded route and a hop-binding secret are included in the invoice.

3. Upfront Payments

   Jamming attacks are possible where an attack can delay a payment resolution and therefore lock bitcoin along a route for a period of time. This attack is described [here](https://lists.linuxfoundation.org/pipermail/lightning-dev/2015-August/000135.html).

   Fidelity Bonds are a solution to

### Payments <a id="d9fec0d4-46ab-4634-ac1c-507a3b5a7a40"></a>

#### Atomic Multi-path Payments \(AMP\) <a id="3b98c12c-454f-4c6e-95d8-2ea818225fb2"></a>

[Mailing List Discussion](https://lists.linuxfoundation.org/pipermail/lightning-dev/2018-February/000993.html)

Atomic multi-path payments allow a single logical payment to be sharded across multiple paths across the network. This is done at the sender side. Multiple subtransactions can be sent over the network and take their own paths and then the necessary constraint is that they all settle together. If one of the payments fails then we want them to all fail. This is where the atomicity comes in. This enables better usage of in-bound and out-bound liquidity. You can send payments over multiple routes, and this allows you to split up and use the network better. This is a more intuitive user experience because like a bitcoin wallet you expect to be able to send most of the money in your wallet. Without AMPs, this is difficult because you can only send a max amount based on current channel liquidity or something, which doesn't really make sense to a user.

Only the sender and receiver need to be aware of this technology.

#### Splicing <a id="fbdef6d1-d18e-48e4-91a4-30eb0ff3b36c"></a>

[Mailing List Discussion](https://lists.linuxfoundation.org/pipermail/lightning-dev/2017-May/000692.html)

With a splice-in, funds can be added to the channel, and they subsume an existing UTXO. And splice-out can remove funds from a participant balance and this creates a UTXO. You can take what's in the channel, splice it off to an address that a person I'm trying to pay controls.

This removes the concept of my "funds are locked in the lightning network" because now you can do both on-chain payments into a channel and you can also do a payment out of a channel without interrupting the normal operation of the channel.

#### Submarine Swap <a id="e8122b54-152b-4449-951b-84886a61a09e"></a>

[SF Bitcoin Devs Talk on Submarine Swaps](https://www.youtube.com/watch?v=ASkyu0w_8Q8)

Submarine swaps allow you to pay someone on-chain and have the payment to them be contingent on them paying a Lightning invoice you give them. If they don't pay the invoice, after a timeout you can get your money back.

#### Hold Invoices <a id="e6e5f788-a391-4c95-bba5-827bd6341635"></a>

[LND implementation](https://github.com/lightningnetwork/lnd/pull/2022)

Instead of immediately locking in and settling the htlc when the payment arrives, the htlc for a hold invoice is only locked in and not yet settled. At that point, it is not possible anymore for the sender to revoke the payment, but the receiver still can choose whether to settle or cancel the htlc and invoice.

#### Trampoline Payments <a id="88cec45e-21f8-4f81-b238-e5a38362f14f"></a>

Lightning network currently relies on source routing where sender calculates the route. Sender needs to maintain graph state.

Trampoline payments is a new suggested way of outsourcing that aims at having lite clients outsourcing the route computation to trampoline nodes, nodes of higher Memory, bandwidth and computation power.

[design decisions on trampoline routing](https://bitcointechweekly.com/front/outsourcing-route-computation-with-trampoline-payments/)

#### Static/Send/Spontaneous/Push Payments <a id="b6b048fe-68df-44d7-b2c8-08fd3eff4ad4"></a>

Wow, lots of names for an overlapping concept.

[OffersStatic Payments](https://github.com/lightningnetwork/lightning-rfc/pull/798)[Push Invoices](https://github.com/lightningnetwork/lightning-rfc/issues/644)

### HTLCs <a id="6666bd59-ae08-4a68-b0d9-18e1af2a74d2"></a>

HTLCs..Hashed Time Lock Contracts.

The initiator of a Lightning channel pays the closing fee. Lots of HTLCs = large fee. See [thread](https://twitter.com/joostjgr/status/1310584596174643200).

[Thread on free HTLC forwarding](https://twitter.com/joostjgr/status/1311608861955158019)

An interesting idea to handling the edge cases around HTLCs is to have a firewall. An [example](https://github.com/lightningequipment/circuitbreaker).

### PTLCs <a id="3485ed65-a52c-44d3-b18f-6269440417dd"></a>

Payment Points[excellent SuredBits blog on PTLCs](https://suredbits.com/payment-points-and-barrier-escrows/)

### Future <a id="2761c42e-2858-4e20-ab0b-23195e647c88"></a>

[challenges and opportunites for ln 2](https://blog.theabacus.io/lightning-network-2-0-b878b9bb356e)

[Why We Fail Lightning](https://medium.com/@antoine.riard/why-we-may-fail-lightning-ee3692de1a55)

### Operations <a id="0a685c5c-6a52-4cd2-b3ef-e50ca28e1f40"></a>

[Node Operation](https://github.com/openoms/lightning-node-management)

The challenges of operating a lightning node deserves its own section. The lightning domain is distinct from on-chain bitcoin due to its own security assumptions, state changes, and end-user experience.

The most immediate concern is backup maintance. With on-chain bitcoin, one can is familiar with BIP39 mnemonic seed phrases as the ultimate backup for bitcoin. In lightning, the backup file is responsible for channels. Do **not** take backups of channel state itself. Inaccurate or revoked channel state is can lead to a justice transaction and punishment \(loss of all funds in the channel\). As a result, backups are tricky in lightning.

Static channel backups \(SCBs\) are the best backups for lightning node operators. The backups are called static because they are only obtained once - when the channel is created. Afterwards, the backup is valid until the channel is closed. A SCB allows a node operator to recover funds that are fully settled in a channel. Fully settled funds are bitcoin in commitment outputs, but not HLTCS.

[LND Recovery Documentation](https://github.com/lightningnetwork/lnd/blob/master/docs/recovery.md)

[LND PR\#2313](https://github.com/lightningnetwork/lnd/pull/2313)

[Automating channel backups for LND](https://gist.github.com/alexbosworth/2c5e185aedbdac45a03655b709e255a3)

[Subscribe to channel backups for LND](https://api.lightning.community/#subscribechannelbackups)

In addition to backups, channel management is a large area of focus. A node operator wants to be connected to reliable and honest peers. Factors to consider are uptime, balance, and cost of rebalancing. It is convenient to create a list of decent nodes and maintain a relationship with them. For inbound liquidity, swaps can be used or swap services like Lightning Labs Loop. Loop can be used to refill channels. Managing incoming channel requests can be important in order to prevent undesirable peers. For example, setting a threshold for channel capacity can prevent dust limit problems in the future. It is better to have fewer channels that are well capitalized than many channels with poor capacity.

Watchtowers can be used to monitor private nodes.

#### Tools
[Moneni](https://moneni.com/)

[LNnodeinsight](https://lnnodeinsight.com/)

[Balance of Satoshis](https://github.com/alexbosworth/balanceofsatoshis)

[charge-lnd](https://github.com/accumulator/charge-lnd)

[rebalance-lnd](https://github.com/C-Otto/rebalance-lnd)

[Node Operator Guide](https://github.com/aljazceru/lightning-network-node-operator)

[Circuit Breaker](https://github.com/lightningequipment/circuitbreaker)

[LNsync](https://medium.com/blockstream/keep-your-node-up-to-date-with-lnsync-e8d8ff7fadb8)
