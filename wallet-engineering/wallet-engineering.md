# Wallet Engineering

## Wallet Engineering <a id="76a407ba-ed00-4579-82a3-1929d952ca74"></a>

_Wallet Design concerns all things related to wallet functionality. This mostly is application level logic._

Modern bitcoin wallets are known as HD wallets or hierarchical deterministic. An HD wallet has a seed and can derive many child keys from a single key. In the early development of bitcoin, wallets would generate a new key for each receive address and then save the key to a file. This unfortunately made backups difficult and error prone. Instead, HD wallets can be backed up using a seed. The familiar 12 or 24 word mnemonic seed phrases are an artifact part of BIP39.

### BIP-32 <a id="45ef83b1-2330-4d6a-9eeb-fa897362fe5c"></a>

Wallets derive a number of child keys from a parent key. To prevent relying on only the key, both private and public keys are extended with an extra 32 bytes of entropy. This entropy is called the chain code.

There are 2^31 child keys and 2^31 hardened child keys. The distinction is very important.

* private parent key -&gt; private child key = computes a child extended private key from the parent extended private key
* public parent key -&gt; public child key = computes a child extended public key from the parent extended public key. It is only defined for non-hardened child keys.
* private parent key -&gt; public child key = computes the extended public key corresponding to an extended private key \(the “neutered” version, as it removes the ability to sign transactions\).
* public parent key -&gt; private child key = not possible

[Deep Dive on Extended Keys](https://bitcoin.stackexchange.com/questions/62533/key-derivation-in-hd-wallets-using-the-extended-private-key-vs-hardened-derivati)

[A Formal Treatment of Deterministic Wallets](https://eprint.iacr.org/2019/698.pdf)

A derivation path is the descriptor for identifying the path along the BIP32 tree.

### BIP-39 <a id="900854ef-205f-46ca-9048-8f57c8c74005"></a>

### Wallet Standards <a id="ea906794-ee9c-4181-8027-f635aa58ae96"></a>

Due to the flexibility of BIP32 trees, standards were created for wallet operators. Standards for the BIP32 tree allows for saner backups and easier portability of seeds between wallet services.

[BIP39 Tool (for testing)](https://iancoleman.io/bip39/)
[Alternative BIP32 Tool (for testing)](https://bip32jp.github.io/english/)

### BIP-43 <a id="f6bfe271-acae-404e-b90e-8fbb7a1e5746"></a>

The first of these standards is [BIP-43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki) which defines the first level of the BIP32 tree as the purpose field.

### BIP-44 <a id="b9b5c913-c2a4-46d5-9ced-85343f5f21d1"></a>

[BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) expands on BIP-43 by specifying the coin and account levels of the BIP32 tree. In addition, the derivation path can describe whether the wallet should derive a change \(or internal\) address or receive \(or external\) address.

### BIP-45 <a id="58945b3a-0bcc-4513-9033-6d750deb5725"></a>

[BIP-45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki)

### BIP-47 <a id="e534d496-cc5a-47a6-a128-fcf64895cbb5"></a>

### SegWit <a id="7b181062-4c33-4419-91bc-b106bb422680"></a>

Since SegWit, couple of changes to wallets were needed:[SegWit wallet dev guide](https://bitcoincore.org/en/segwit_wallet_dev/)

One of the immediate problems that SegWit solves is mitigating transaction malleability.

### Covenants <a id="0a6ed404-ea36-4dba-a840-8b958f714aac"></a>

A covenant is a bitcoin script that restricts the type of transaction that can spend a coin.

[Greg Maxwell’s Bitcoin Talk thread on covenants](https://bitcointalk.org/index.php?topic=278122.0)

[OP\_CAT and Schnorr - Part 1](https://medium.com/blockstream/cat-and-schnorr-tricks-i-faf1b59bd298)

[OP\_CAT and Schnorr - Part 2](https://www.wpsoftware.net/andrew/blog/cat-and-schnorr-tricks-ii.html)

### Vaults <a id="fdc29a1c-360e-4bf3-b97c-92c2092e752d"></a>

[Vaults without Covenants](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-August/017229.html)[more by bishop](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-August/017231.html)

[Proof of Reserves by bBlockstream](https://blockstream.com/2019/02/04/en-standardizing-bitcoin-proof-of-reserves/)

[Making Bitcoin Exchanges Transparent](https://tik-old.ee.ethz.ch/file//b89cb24ad2fa4e7ef01426d318c9b98b/decker2015making.pdf)

BIP-127 proposes a standard way to do proof of reserves using a PSBT extension.[link to bip](https://github.com/bitcoin/bips/blob/master/bip-0127.mediawiki)

There’s rust implementation of a Proof-of-Reserves Client. [link to reserves](https://github.com/ElementsProject/reserves)

[custody protocols using bitcoin vaults](https://arxiv.org/pdf/2005.11776.pdf)

### Batching <a id="697eb869-348c-4f1f-b803-4e09cae91cd6"></a>

Payment batching, more [here](https://github.com/bitcoinops/scaling-book/blob/master/x.payment_batching/payment_batching.md), is including multiple payments inside a single transaction.

Variables to consider are \# of inputs and \# of outputs. Better to have a single input and many outputs. It is also nice to have a lower fee for the entire transaction.

Goal of batching is to lower vbytes per payment. Marginal improvmenent after 1 input and 5 outputs.

### Coin Selection <a id="d989d56d-f60b-4898-a122-016a6f2678db"></a>

[Challenges of coin selection by Lopp](https://medium.com/@lopp/the-challenges-of-optimizing-unspent-output-selection-a3e5d05d13ef)

[Coin Selection with Leverage](https://arxiv.org/pdf/1911.01330.pdf)

[IOHK on Coin Selection](https://iohk.io/en/blog/posts/2018/07/03/self-organisation-in-coin-selection/) 

[What is Coin Selection?](https://bitcoin.stackexchange.com/questions/1077/what-is-the-coin-selection-algorithm)

[Murch transcript at Scaling](https://diyhpl.us/wiki/transcripts/scalingbitcoin/milan/coin-selection/)


[Edge++ Coin Selection transcript](http://diyhpl.us/wiki/transcripts/scalingbitcoin/tokyo-2018/edgedevplusplus/coin-selection/)

The naive approach would be to simply look for the smallest output that is larger than the amount you want to spend and use it, otherwise start adding the next largest outputs until you have enough outputs to meet the spend target. However, this leads to fracturing of outputs until the wallet becomes littered with unspendable “dust.”

“Our idea is to have the user the option \(either global or per account or per transaction\) to choose between”maximize privacy" or “minimize fees” \(or even maybe “minimize UTXO”

”Dust” refers to transaction outputs that are less valuable than three times the mininum transaction fee and are therefore expensive to spend.

A transaction output is labeled as dust when its value is similar to the cost of spending it. Precisely, Bitcoin Core sets the dust limit to a value where spending an 2.3. Transactions 7 output would exceed 1/3 of its value. This calculation is based on the minimum relay transaction fee, a node setting that causes transactions that don’t at least include this lower bound of fee to be dropped from the memory pool, and not relayed to other nodes. With the default for the minimum relay transaction fee set to 1 000 satoshi per kilobyte, and the sizes of a P2PKH input being 148 bytes, and an output being 34 bytes, this computes to all outputs smaller or equal to 546 satoshis being considered dust by Bitcoin Core \[Erha15\].

[utxo mgmt for enterprise wallets](https://blog.bitgo.com/utxo-management-for-enterprise-wallets-5357dad08dd1)

### Bitcoin Core Wallet <a id="5adac067-7dd4-4377-90d7-76f7f9a9299e"></a>

Bitcoin Core’s wallet is always evolving. Some changes to the Bitcoin Core wallet: [Wallet Class Structure Changes](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/Wallet-Class-Structure-Changes) [Sipa describing wallet changes](https://gist.github.com/sipa/125cfa1615946d0c3f3eec2ad7f250a2) [Wallet Architecture transcripts](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2019-06-05-wallet-architecture/) [What’s Coming to Bitcoin Core Wallet in 2021](https://achow101.com/2020/10/0.21-wallets) [Descriptor Wallets MD](https://github.com/bitcoin/bitcoin/blob/master/doc/release-notes.md#experimental-descriptor-wallets)

### Descriptors <a id="2ba96219-33f7-4625-97db-98f706318eac"></a>

[Bitcoin Issue 17190](https://github.com/bitcoin/bitcoin/issues/17190) [Electrum on Descriptors](https://github.com/spesmilo/electrum/issues/5715) [Descriptors Overview](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md) [coredev talk](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2018-10-08-script-descriptors/)

Implementations… [HWI](https://github.com/bitcoin-core/HWI/blob/95c9387215fd534bb7a7e3e1885d92cc22457847/hwilib/descriptor.py) [Bitcoin \#16528](https://github.com/bitcoin/bitcoin/pull/16528) [Bitcoin Core parsing](https://github.com/bitcoin/bitcoin/blob/befdef8aee899dcf7e40aa5ea4bc1b0256381cdc/src/util/spanparsing.cpp) [Bitcoin \#15764](https://github.com/bitcoin/bitcoin/pull/15764)

### Script <a id="765c5857-2a97-4ebe-9a54-49e8fa134eae"></a>

\([https://en.bitcoin.it/wiki/Contract](https://en.bitcoin.it/wiki/Contract)\) Each transaction input has a sequence number. In a normal transaction that just moves value around, the sequence numbers are all UINT\_MAX and the lock time is zero. If the lock time has not yet been reached, but all the sequence numbers are UINT\_MAX, the transaction is also considered final.

Sequence numbers can be used to issue new versions of a transaction without invalidating other inputs signatures, e.g., in the case where each input on a transaction comes from a different party, each input may start with a sequence number of zero, and those numbers can be incremented independently.

Signature checking is flexible because the form of transaction that is signed can be controlled through the use of SIGHASH flags, which are stuck on the end of a signature. In this way, contracts can be constructed in which each party signs only a part of it, allowing other parts to be changed without their involvement. The SIGHASH flags have two parts, a mode and the ANYONECANPAY modifier:

1. SIGHASH\_ALL: This is the default. It indicates that everything about the transaction is signed, except for the input scripts. Signing the input scripts as well would obviously make it impossible to construct a transaction, so they are always blanked out. Note, though, that other properties of the input, like the connected output and sequence numbers, are signed; it’s only the scripts that are not. Intuitively, it means “I agree to put my money in, if everyone puts their money in and the outputs are this”.
2. SIGHASH\_NONE: The outputs are not signed and can be anything. Use this to indicate “I agree to put my money in, as long as everyone puts their money in, but I don’t care what’s done with the output”. This mode allows others to update the transaction by changing their inputs sequence numbers.
3. SIGHASH\_SINGLE: Like SIGHASH\_NONE, the inputs are signed, but the sequence numbers are blanked, so others can create new versions of the transaction. However, the only output that is signed is the one at the same position as the input. Use this to indicate “I agree, as long as my output is what I want; I don’t care about the others”.

There are two general patterns for safely creating contracts:

1. Transactions are passed around outside of the P2P network, in partially-complete or invalid forms.
2. Two transactions are used: one \(the contract\) is created and signed but not broadcast right away. Instead, the other transaction \(the payment\) is broadcast after the contract is agreed to lock in the money, and then the contract is broadcast.

This is to ensure that people always know what they are agreeing to. Together, these features let us build interesting new financial tools on top of the block chain.

It may even be that people find themselves working for the programs because they need the money, rather than programs working for the people.

old oracle services…[https://docs.oraclize.it/\#home](https://docs.oraclize.it/#home)[http://orisi.org/](http://orisi.org/)[http://earlytemple.com/](http://earlytemple.com/)[https://en.bitcoin.it/wiki/Contract\#Example\_4:\_Using\_external\_state](https://en.bitcoin.it/wiki/Contract#Example_4:_Using_external_state)

[SuredBits’ blog on scriptless scripts](https://suredbits.com/schnorr-applications-scriptless-scripts/)

[Poelstra ppt](https://download.wpsoftware.net/bitcoin/wizardry/mw-slides/2018-05-18-l2/slides.pdf)

[Scriptless Scripts](http://diyhpl.us/wiki/transcripts/grincon/2019/scriptless-scripts-with-mimblewimble/)

### Fee Estimation <a id="62f89fbd-8f4a-4c53-a4ca-90bf68abb12c"></a>

[lopp on fee estimation](https://blog.bitgo.com/the-challenges-of-bitcoin-transaction-fee-estimation-e47a64a61c72)

Fee estimation is the process of estimating a particular fee rate to use for a transaction in order to incentivize block inclusion at a particular block target.

Supply \(blocks\) and demand \(txns\) are unpredicable.

[John Newbery’s intro to Bitcoin Core Fee Estimation](https://bitcointechtalk.com/an-introduction-to-bitcoin-core-fee-estimation-27920880ad0) [pt2](https://bitcointechtalk.com/whats-new-in-bitcoin-core-v0-15-part-2-41b6d0493136)

1. Outline of Newbery’s post

   At broadcast, the transaction is not going to get into the next block. But rather likely the next block in 10 minutes. Block production follows Poisson distribution.

   As a result, the fee rate should be competitive not only of the current mempool but the likely mempool in ten minutes.

   Looking only at mempool does not consider lucky block runs.

2. Bitcoin Core’s Fee Estimation Also, the following is recorded:

   [High level desc of Bitcoin Core’s fee estimation algorithm](https://gist.github.com/morcos/d3637f015bc4e607e1fd10d8351e9f41)[code](https://github.com/bitcoin/bitcoin/blob/master/src/policy/fees.h) Bitcoin core groups transaction fee rates into buckets. Each buck is a range of fee rates. A track of block targets from 1 block to 1008 blocks is kept.

   1. number of transactions that entered the mempool in each fee rate bucket.
   2. for each bucket-target pair, the number of transactions that were included in a block within the target number of blocks.

   For any target-bucket pair, Bitcoin Core can find the probability that a transaction with the fee rate can be included. This is B/A.

   [Additional overview](https://blog.iany.me/2020/08/bitcoin-core-fee-estimate-algorithm/)

3. Mempool File Format

   Mempool File Format can be useful for fee estimation..[talk by kalle](https://bc-2.jp/bb2019-mempool-analysis-simulation.pdf) Time series of a txn lifecyle until block inclusion in a small file format.

   [https://github.com/kallewoof/mff](https://github.com/kallewoof/mff)

4. Light Clients

   [Using AI for Fee Estimation](https://bitcoindevkit.org/blog/2021/01/fee-estimation-for-light-clients-part-1/)

### Backup and Recovery <a id="50ce70bc-5556-4c7d-87a1-f1380502f436"></a>

#### BIP39 <a id="e1382bc5-d869-420b-a433-6f8f9859c387"></a>

US NIST recommends a salt length of 128 bits. Passphrase entropy in bits is log2\(\# of combinations\).

BIP39 Wordlist is 2048 word dictionary. So log2\(2048^s\) = 22 bits of entropy.

#### Shamir’s Secret Sharing <a id="5ceca4b1-2efb-4267-b191-2bc04b36e555"></a>

[3](about:blank#fn.3)Shamir’s Secret Sharing \(SSS\) is an algorithm to divide a secret into parts that can be recovered by combining a minimum number of parts.

Each part is called a share. The threshold is the number of shares that is needed to recover the original secret. There can be more shares than the threshold.

There are several criticisms of SSS.

1. [Shamir Secret Snakeoil](https://en.bitcoin.it/wiki/Shamir_Secret_Snakeoil)
2. [Shamir’s Secret Sharing Shortcomings](https://blog.keys.casa/shamirs-secret-sharing-security-shortcomings/)

[Sharding vs Multisig?](https://bitcoin.stackexchange.com/questions/102007/is-sharding-a-good-alternative-to-multisig)

### Light Clients
[SPV Flowchart](https://cloud.githubusercontent.com/assets/2216012/9577839/280ba9fa-500f-11e5-9a18-b879e319389b.png)


