# Protocol

## Protocol <a id="27e73f89-d634-4e34-a3f5-1b471ca32b1d"></a>

_An overview of the Bitcoin protocol and the building blocks of Bitcoin._[Protocol Documentation](https://en.bitcoin.it/wiki/Protocol_documentation)

Ultimately, the behavior of Bitcoin is defined by codebase of the reference client. This is not a complete survey of the bitcoin protocol, but a piecemeal overview of different components that make up Bitcoin.

The reference client, Bitcoin Core, was originally written by Satoshi, but has since been an open source project. The C++ client is the most widely used implementation of Bitcoin.[1](about:blank#fn.1) One of the primary reasons to run the Bitcoin Core client is to stay in consensus with the rest of the network. There is no formal specification of consensus, so consensus is adopted by running the reference client. There are several noteable alternative implementations of Bitcoin, but their usage as consensus daemon are limited. An argument against adopting other implementations is the risk of breaking consensus with the network.

### Primitives <a id="0503f734-82d6-4f6e-95d0-b5502d23a53f"></a>

#### Hashes <a id="7e7ad24d-5a7c-4620-9e86-38227eecf4e9"></a>

Hashes are one-way functions that accept an input and return a fixed-length value. For the hash function to have integrity, the value returned must be unique. As a result, a strong hash function will make it unlikely that two different inptus have the same output. This is called a hash collision.

In Bitcoin, SHA-256 and RIPEMD-160 are used. It is common to see double hashing where an input is hashed twice.

#### Merkle Trees <a id="0688a0f9-3fb6-41f6-afc7-048e21d417f1"></a>

A merkle tree is a binary tree of hashes. It is a container that hashes items and concatenates the hashes in such a way that a final merkle root of the tree is a hash. This merkle root represents the the hashes and concatenation of all the items in the tree.

In Bitcoin, merkle trees are used in a variety of ways. Most notably, a block will have a merkle root of transactions. An excellent utility of merkle trees is proving that a leaf is part of the tree.

#### Signatures <a id="0974d88e-6e3c-4d16-a2f4-6587fdd5c9dc"></a>

Digital signatures are used by a verifier to prove that a signature belongs to a public key. Signatures can only be created using a private key corresponding to the public key.

Bitcoin uses ECDSA, or Ellipitic Curve Digital Signature Algorithm, to sign for transactions and messages. The curve used is secp256k1.

#### Addresses <a id="5a01119a-08fb-4dd9-8395-2ade3bb8725a"></a>

Bitcoin addresses are a human readable format to represent a Bitcoin script. Addresses can be shared to others to receive payment. They should be treated as single-use tokens.

#### Transactions <a id="eb0453fc-d5cd-4ba9-99a4-d0379c76aaa6"></a>

#### Blocks <a id="b41b852c-3908-4de2-a9bb-e9f88887dcfa"></a>

### Consensus <a id="06ca2726-6c89-4828-93a8-aebfdd643bc5"></a>

The consensus of Bitcoin is a large topic and part of extensive research.

A quick glimpse into how difficult it is to describe consensus can be found in David Harding’s [Bitcoin Paper Errata and Details](https://gist.github.com/harding/dabea3d83c695e6b937bf090eddf2bb3) document. The expectations of the software developer often does not match the reality of the code written. Ultimately, consensus is defined by the software that people choose to run on their machines. This software can have bugs and these bugs are technically part of consensus.

There is also social consensus, the ideas and narratives that people choose to adopt and identify with. Social consensus can drive technical consensus to adopt new rulesets.

### Script <a id="90053964-efd7-495a-86ad-05b061da5c86"></a>

### Segregated Witness <a id="b06eb81f-df34-4239-8610-f6d4e81118da"></a>

### Taproot/Schnorr <a id="eef05ced-749c-4959-96ae-4012b5bd3a32"></a>

_Notes on BIP340-342. All things concerning Schnorr, Taproot, and Tapscript._

#### Introduction <a id="2846bee3-b198-4fb7-99e8-25a1b2cef2b4"></a>

[Slides from Sipa's talk at SF Bitcoin Devs](https://prezi.com/view/AlXd19INd3isgt3SvW8g/)

[Taproot Workshop](https://github.com/bitcoinops/taproot-workshop)

[Taproot Review](https://github.com/ajtowns/taproot-review)

immediate benefit of taproot: “if you lose this key, your funds are gone” to “if you lose this key, you’ll have to recover 3 of your 5 backup keys that you sent to trusted friends, and pay a little more, but you won’t have lost your funds”" - anthony towns

Notation to be used throughout \(from BIP\):

* hashtag\(x\) notation to refer to SHA256\(SHA256\(tag\) \|\| SHA256\(tag\) \|\| x\)
* q is taproot output key
* p is taproot internal key

#### BIP340 <a id="8489fc23-d7fa-4c5d-8f3b-21ca0d2910e6"></a>

[Link to BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)

[Why only x-pubkey?](https://medium.com/blockstream/reducing-bitcoin-transaction-sizes-with-x-only-pubkeys-f86476af05d7)

Proposes standard for 64-byte Schnorr signatures.

Why Schnorr?

* Provable security Security of ECDSA rely on stronger assumptions. Schnorr signatures are provably secure.
* Non-malleability Schnorr signatures are implied to be non-malleable given SUF-CMA security. ECDSA sigs are inherently malleable.
* Linearity

Encoding: Instead of DER, use fixed 64-byte format. Public Key Encoding: Instead of compressed 33-byte key, use 32 bytes.

Interestingly, the aim of the BIP is to have the Schnorr spec completely specified. In the past, different ECDSA implementations caused issues.

[SuredBits’ intro to Schnorr](https://suredbits.com/introduction-to-schnorr-signatures/)

“tweaking” involves hiding/obfuscation

[security of blind discrete log signatures](https://www.math.uni-frankfurt.de/~dmst/research/papers/schnorr.blind_sigs_attack.2001.pdf)

[generalized bday problem](https://www.iacr.org/archive/crypto2002/24420288/24420288.pdf)

[x-only pubkeys](https://medium.com/blockstream/reducing-bitcoin-transaction-sizes-with-x-only-pubkeys-f86476af05d7)

[lattice attacks against weak ECDSA](https://eprint.iacr.org/2019/023.pdf)

A schnorr signature is defined as the following: S = R + H\(x\(R\)\|P\|m\) \* P where R is the Nonce point \(k\*G\)

To save 32 bytes, only the x value of R is provided by the signer. The verifier can computer the y-value.

One of the y-coordinates is even while the other is odd.

Proposal constraints k such that y-value of R is quadratic residue module SECP256K1\_FIELD\_SIZE. Quadaratic residue is having a square root modulo the field size.

If a randomly generated nonce k does not yield a valid nonce point R, then the signer can negate k to obtain a valid nonce.

#### BIP341 <a id="463570d2-24f1-4ac7-9d7e-e2ea8d974509"></a>

[Link to BIP341](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)

Proposes new SegWit v1 output type with spending rules based on Taproot, Schnorr, and merkle branches.

BIP claims no new security assumptions are added.

The aims of the output type is to improve privacy, efficiency, and flexibility of Bitcoin script. This is especially useful in minimizing how much information is shown on the blockchain regarding the spendability conditions. Additionally, a few bug fixes are included.

The BIP is very selective in the technologies that are included. Many are swept for later review in order to reduce complexity of review as well as prevent immature technology from weighind down ready technology.

From the BIP document, the following technologies compose the proposal:

* Merkle Branches: Reveal the actual executed part of the script.
* Taproot: Merge pay-to-pubkey and pay-to-scripthash policies making outputs spendable by either indistiguishable. As long as key-based spending path is used for spending, it is not revealed whether a script path was permitted as well. An assumption is made that most outputs can be spent by all parties agreeing. Schnorr permits key aggregation.

  [2](about:blank#fn.2)

Key aggregation allows a public key to be constructed from multiple participant keys. Indistinguishable from single-party.

* Batch validation is permited with schnorr signatures.
* Every merkle tree has an associated version allowing for new script versions to be introduced via soft fork. Unused ‘annex’ in the witness can also be used.
* New Signature Hashing Algorithm includes amount and ScriptPubKey in message. And uses tagged hashes.
* The public key is directly included in the output.

BIP can be informally summarized in the following way:

```text
a new witness version is added (version 1), whose programs consist of 32-byte encodings of points Q. Q is computed as P + hash(P||m)G for a public key P, and the root m of a Merkle tree whose leaves consist of a version number and a script. These outputs can be spent directly by providing a signature for Q, or indirectly by revealing P, the script and leaf version, inputs that satisfy the script, and a Merkle path that proves Q committed to that leaf. All hashes in this construction (the hash for computing Q from P, the hashes inside the Merkle tree's inner nodes, and the signature hashes used) are tagged to guarantee domain separation.
```

A taproot output is a native SegWit output with version number 1 and a 32-byte witness program.

Every taproot output corresponds to a combination of a single public key condition \(internal key\), and zero or more general conditions encoded in scripts in a tree.

General guidelines for construction and spending Taproot outputs:

* Better to split scripts with conditionls \(OP\_IF\) into multiple scripts in the tree…each corresponding to one execution path.
* When a single condition requires signautres from multiple keys, key aggregation MuSig can be used.
* Most likely key to be used should be the internal key. If no such condition exists, worthwhile adding one that consists of an aggregation of all keys. This is an “everyone agrees” branch. Else just pick an internal key using a point wi unknown discrete logarithm. See BIP for example.
* If no script conditions needed, an output key should commit to an unspendable script path instead. See BIP for how to achieve this.
* Remaining scripts should be organized into leaves. Huffman tree.
* Binary tree leaves are \(leaf\_version, script\) tuples.

Q=P+H\(P,m\)\*G where P is public key and m is merkle root of a MAST.

[switchable scripting](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html)

#### BIP342 <a id="70fa74b2-bb61-4e8a-b899-b5a3edbd253f"></a>

[Link to BIP342](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki)

Proposes semantics of the scripting system described in BIP341.

Includes improvements to schnorr signatures, batch validation, and signature hash.

OP\_CHECKSIG and OP\_CHECKSIGVERIFY are modified to verify schnorr signatures. OP\_CODESEPARATOR simplified.

OP\_CHECKMULITSIG and OP\_CHECKMULTISIGVERIFY are disabled. OP\_CHECKSIGADD is introduced to make multisigs batch-verifiable.

A potential malleability vector is eleminated by requiring MINIMALIF. Using a non-standard represetentation of true for OP\_IF is now considered invalid as a violation of consensus rules.

OP\_SUCCESS opcodes allows introducing new opcodes cleanly than through OP\_NOP.

Tapscript can be upgraded through soft forks by defining unknown key types. For example, adding a new hash\_types or signature algorithms.

#### MuSig <a id="8d8e02e3-2282-4166-8db7-6d87edc61db6"></a>

Schnorr multi-signature scheme.

Blockstream announcing [MuSig.](https://blockstream.com/2019/02/18/en-musig-a-new-multisignature-standard/)

[actual whitepaper](https://eprint.iacr.org/2018/068.pdf)

[SuredBits’ blog on musig](https://suredbits.com/schnorr-applications-musig/)

[Insecure Shortcuts in MuSig](https://medium.com/blockstream/insecure-shortcuts-in-musig-2ad0d38a97da)

[MuSig-DN: Deterministic Nonces](https://medium.com/blockstream/musig-dn-schnorr-multisignatures-with-verifiably-deterministic-nonces-27424b5df9d6)

[MuSig Interactivity](https://bitcoin.stackexchange.com/questions/91534/musig-signature-interactivity)

[The Soundness of MuSig](https://joinmarket.me/blog/blog/the-soundness-of-musig/)

1. MuSig2

   Exchanging nonce commitments is the subject of the [MuSig-DN paper](https://medium.com/blockstream/musig-dn-schnorr-multisignatures-with-verifiably-deterministic-nonces-27424b5df9d6).

   Nonce commitment exchange can be removed by generating the nonce deterministically from the signers’ public keys and message. Providing a non-interactive zk proof that the nonce was generated deterministically along with the nonce.

   The MuSig2 scheme has a two round signing protocol w/o the need for a sk proof. Also, the first round of the nonce exchange is done at key setup time.

   Therefore, there are two variants: interactive setup and non-interactive setup.

   [BitcionOps explains MuSig2](https://bitcoinops.org/en/newsletters/2020/10/21/)

   [MuSig2](https://eprint.iacr.org/2020/1261)

#### SIGHASH\_ANYPREVOUT <a id="513ed7dd-7801-4402-a6d9-59771727a562"></a>

[proposed bip](https://github.com/ajtowns/bips/blob/bip-anyprevout/bip-anyprevout.mediawiki)

a new type of public key for tapscript \(bip-tapscript\) transactions. It allows signatures for these public keys to not commit to the exact output being spent. This enables dynamic binding of transactions to different UTXOs, provided they have compatible scripts.

Allows dynamic rebinding of a signed transaction to another previous output of the same value

### Mining <a id="b44f0e97-68bd-4b14-b470-3994b0fed56a"></a>

_All things Bitcoin mining._

#### Introduction <a id="71c6ee36-69b2-4068-b86c-6dca87668230"></a>

[Excellent podcast on mining](https://stephanlivera.com/episode/128/)

cgminer is open source miner for ASIC/FPGA miner. Lots of companies forked off this original miner.[https://github.com/ckolivas/](https://github.com/ckolivas/)

[Channel payouts in mining](https://bitcointalk.org/index.php?topic=2135429.msg21352028)

[The Problem with ASICBOOST](http://www.mit.edu/~jlrubin//public/pdfs/Asicboost.pdf)

[Optimal pool abuse strategy](http://bitcoin.atspace.com/poolcheating.pdf)

[Analysis of Bitcoin Pooled Mining Reward Systems](https://sites.cs.ucsb.edu/~rich/class/cs293b-cloud/papers/bitcoin-pool.pdf)

[Characterizing Orphan Transactions](https://arxiv.org/pdf/1912.11541.pdf)

#### GetBlockTemplate <a id="6b1e788a-a4ec-49c3-b548-679b0c4778bd"></a>

Getblocktemplate: bitcoin core &lt;-&gt; pool server

#### Stratum <a id="66efa13c-3a79-4e4b-b975-b902ef2ef296"></a>

Stratum: pool server &lt;-&gt; asic controller

[Stratum Protocol documentation](https://slushpool.com/help/topic/stratum-protocol/)

The design of the Stratum protocol requires pool operators to build and distribute block templates to their clients.

#### StratumV2 <a id="a8ca697d-b8fd-4425-87af-904f1515ca51"></a>

[StratumV2 Migration](https://insights.deribit.com/market-research/stratum-v2-migration-and-decentralization/)

#### Betterhash <a id="083f1957-2713-4ec7-8ba9-25ff6db4ea3a"></a>

* Work protocol: bitcoin
* core &lt;-&gt; mining proxy
* Work protocol: mining proxy/bitcoin core &lt;-&gt; asic controller
* Pool protocol: pool server &lt;-&gt; mining proxy

[link to bip](https://github.com/TheBlueMatt/bips/blob/betterhash/bip-XXXX.mediawiki)

[betterhash overview](https://medium.com/hackernoon/betterhash-decentralizing-bitcoin-mining-with-new-hashing-protocols-291de178e3e0)

#### Compact Blocks <a id="e7695c05-4a96-42fb-a57e-27e3c93d3aa2"></a>

[faq for compact blocks](https://bitcoincore.org/en/2016/06/07/compact-blocks-faq/) Compact block relay, BIP152, is a method of reducing the amount of bandwidth used to propagate new blocks to full nodes.

Using simple techniques it is possible to reduce the amount of bandwidth necessary to propagate new blocks to full nodes when they already share much of the same mempool contents. Peers send compact block “sketches” to receiving peers.

### P2P <a id="5c63eeca-27ae-4409-963a-67f7126576dd"></a>

_P2P layer of Bitcoin._ For the Bitcoin network to remain in consensus, the network of nodes must not be partitioned. So for an individual node to remain in consensus with the network, it must have at least one connection to that network of peers that share its consensus rules.

[Partition Resistance](https://gist.github.com/sdaftuar/c2a3320c751efb078a7c1fd834036cb0)

[Eclipse Attacks on Bitcoin’s Peer-to-Peer Network](https://eprint.iacr.org/2015/263.pdf)

