# Other

_Notes that do not fit neatly in the other categories._

Merkelized Abstract Syntax Trees are a general concept: when bitcoin developers talk about it, they’re talking about reworking bitcoin scripts into a series of “OR” branches, and instead of the output committing to the whole script, you commit to the head of the tree.  To spend it, you only need to provide the branch of the script you’re using, and the hashes of the other branches. This can improve privacy, and also shrink the total size of large scripts, particularly if there’s a short, common case, and a long, complex rare case. Note that each key is 33 bytes and each signature about 72 bytes, and each merkle branch only 32 bytes.

Sidechains are based on cross-chain consensus validation through SPV and reorganization proofs \(an idea that dates back to my P2PTradeX protocol\), while drivechains are based on miners being consensus proxies.

The idea behind JoinMarket is to help create a special kind of bitcoin transaction called a CoinJoin transaction. It’s aim is to improve the confidentiality and privacy of bitcoin transactions, as well as improve the capacity of the blockchain therefore reduce costs. The concept has enormous potential, but had not seen much usage despite the multiple projects that implement it. This is probably because the incentive structure was not right. A CoinJoin transaction requires other people to take part. The right resources \(coins\) have to be in the right place, at the right time, in the right quantity. This isn’t a software or tech problem, its an economic problem. JoinMarket works by creating a new kind of market that would allocate these resources in the best way.

Merged mining is the act of using work done on another block chain \(the Parent\) on one or more Auxiliary block chains and to accept it as valid on its own chain, using Auxiliary Proof-of-Work \(AuxPoW\), which is the relationship between two block chains for one to trust the other’s work as their own. The Parent block chain does not need to be aware of the AuxPoW logic as blocks submitted to it are still valid blocks.

### Future directions of bitcoin <a id="12c20179-6e1f-4767-9b37-c8971ce744a2"></a>

[transcript](http://diyhpl.us/wiki/transcripts/2018-01-24-rusty-russell-future-bitcoin-tech-directions/) Schnorr Signature Scheme

* Has security proof, EDCSA does not.
* Has linear property, sum of sigs is sum of keys.

SIGHASH\_NOINPUT - sign scripts, not txid

Taproot basic idea-&gt; tweak pubkey Q = P+H\(P,S\)G Q in output key spend sign\(Q\) script spend: P,S, inputs

Graftroot if a key exists to represent everyone use delegation instead of merkle tree inherently interactive key setup

### Utreexo <a id="1ff04b0b-af3e-4bdf-8ca3-b6f5e25a1cdc"></a>

[accumulators](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2018-10-08-utxo-accumulators-and-utreexo/)

### Graftroof <a id="ead4058a-ede2-4c04-8093-1ef9c95bd64b"></a>

The idea of graftroot is that in every contract there is a superset of people that can spend the money. In graftroot, if all the participants agree, then they can just spend. So they can do pubkey aggregation on P

Taproot: P = c + H\(c \|\| script\) G

Graftroot: sigp\(script\)

[graftroot vs taproot](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2018-03-06-taproot-graftroot-etc/)

### AssumeUTXO <a id="82436512-8310-4207-8985-51303994ecf6"></a>

You get a serialized UTXO set snapshot obtained by a peer. This all hinges on a content-based hash of the UTXO set. The peer gets headers chain, ensures base of snapshot in chain, load snapshot. They want to verify the base of the snapshot or the blockhash is in the header chain. We load the snapshot which deserializes a bunch of coins and loads it into memory. Then we fake a blockchain; we have a chainstate but no blocks on disk, so it’s almost like a big pruned chain. We then validate that the hash of the UTXO set matches what we expected through some hardcoded assumeutxo. This is a compiled parameter value, it can’t be specified at runtime by the user which is very important. At that point, we sync the tip and that will be a similar delta to what assumevalid would be now, maybe more frequent because that would be nice. Crucially, we start background verification using a separate chainstate where we do regular initial block download, bnackfill that up to the base of the snapshot, and we compare that to the hash of the start of the snapshot and we verify that.[talk on assumeutxo](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2019-06-07-assumeutxo/)

[bitcoin-dev email](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-April/016825.html)

The initializing node syncs the headers chain from the network, then obtains and loads one of these UTXO snapshots \(i.e. a serialized version of the UTXO set bundled with the block header indicating its “base” and some other metadata\).

hardcoded hashs exist in software ..hash\(utxoset\). similar to assumevalid.

snapshots can obtained in same manner as block download. Doesn’t matter about source cuz of content hash.

### CoinWitness <a id="a0d6074f-9045-4d45-a07a-5db2e1b0bbf3"></a>

Applications of ZK Snarks… “. Instead of embedding the rules that govern an output inside the blockchain, you’d instead embed a proof that the rules were followed. Instead of everyone checking that a transaction was permitted to be spent, they’d instead check that you checked.” - Maxwell

[coin witness](https://bitcointalk.org/index.php?topic=277389.0)

“You write down a small program which verifies the faithfulness of one of these transcripts for your chosen verifiable off-chain system. The program requires that the last transaction in the transcript is special in that it pays to a Bitcoin scrippubkey/p2sh. The same address must also be provided as a public input to the program. We call this program a”witness" because it will witness the transcript and accept if and only if the transcript is valid.

You then use the SCIP proof system to convert the program into a verifying key. When someone wants to create a Bitcoin in an off-chain system, they pay that coin to the hash of that verifying key. People then transact in the off-chain system as they wish. To be confident that the system works faithfully they could repeat the computationally-expensive verifying key generation process to confirm that it corresponds to the transaction rules they are expecting.

When a user of one of these coins wants to exit the system \(to compact its history, to move to another system, to spend plain Bitcoins, or for any other reason\), they form a final transaction paying to a Bitcoin address, and run the witness on their transcript under SCIP and produce a proof. They create a Bitcoin transaction redeeming the coin providing the proof in their script \(but not the transcript, thats kept private\), and the Bitcoin network validates the proof and the transaction output. The public learns nothing about the intermediate transactions, improving fungibility, but unlike other ideas which improve fungibility this idea has the potential to both improve Bitcoin’s scalability and securely integrate new and innovative alternative transaction methods and expand Bitcoin’s zero-trust nature to more types of transactions."

### Covenants <a id="6d2de0ef-f964-4ac4-8088-e668052d479e"></a>

A covenant in its most general sense and historical sense, is a solemn promise to engage in or refrain from a specified action.[maxwell on covenants](https://bitcointalk.org/index.php?topic=278122.0)

[scaling bitcoin](https://diyhpl.us/wiki/transcripts/scalingbitcoin/milan/covenants/) “Covenants can be recursively enforced down the chain for as long as you need to reinforce them.”

“Covenants can be used to break fungibility.”[whitepaper](https://fc16.ifca.ai/bitcoin/papers/MES16.pdf)

[Bitcoin Covenants: Three Ways to Control the Future](https://arxiv.org/abs/2006.16714)

### Zero Knowledge Contigent Payment <a id="75958c39-2491-4cf6-aa27-8f3c1d5215ef"></a>

[zero knowledge payment](https://bitcoincore.org/en/2016/02/26/zero-knowledge-contingent-payments-announcement/) ZKCP

swapping information for value

### Discreet Log Contracts <a id="03a91ead-ed13-4cad-b93d-f80abae9b266"></a>

[Whitepaper](https://adiabat.github.io/dlc.pdf)[DLC Spec](https://github.com/discreetlogcontracts/dlcspecs)

