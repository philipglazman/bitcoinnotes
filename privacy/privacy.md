# Privacy

## Privacy <a id="0f7882f3-7cfc-4d28-a618-13b2351ea099"></a>

_Privacy and techniques used in chain-analysis._

[Privacy Wiki](https://en.bitcoin.it/wiki/Privacy)

[snowball presentation at ldn bitdevs](https://www.youtube.com/watch?v=peT_9XF2L04)

common input hueristic: “different public keys used as inputs to a transaction as being controlled by the same user”[original paper on blockchain analysis](https://cseweb.ucsd.edu/~smeiklejohn/files/imc13.pdf)

[coin join wiki](https://github.com/6102bitcoin/CoinJoin-Research)

[Bitcoin vs “privacy coins”](https://bitcoin.stackexchange.com/questions/101868/is-there-something-about-bitcoin-that-prevents-us-from-implementing-the-same-pri)

### CoinJoins <a id="90b5c3fb-b8a5-490d-8a81-115c93a7b9d4"></a>

“So a world where”basically everyone uses CoinJoin" is cool for privacy, but could end up pretty bad for scalability, because these transactions are in addition to the normal payments." - waxwing

1. PayJoin

   [payjoin by waxwing](https://joinmarket.me/blog/blog/payjoin/) PayJoin is coinjoin + payment

   “Let Bob do a CoinJoin with his customer Alice - he’ll provide at least one utxo as input, and that/those utxos will be consumed, meaning that in net, he will have no more utxos after the transaction than before, and an obfuscation of ownership of the inputs will have happened without it looking different from an ordinary payment.”

   “the main point is with PayJoin - we break the heuristic without flagging to the external observer that the breakage has occurred.” … unlike coinjoins

   “snowball effect” … payjoin/p2ep reduces utxo set and receiver’s utxo gets bigger after each payment txn.

   who pays for the fee? “every payment to the merchant creates a utxo, and every one of those must be paid for in fees when consumed in some transaction.”

   real world implementation is [samourai wallet](https://samouraiwallet.com/stowaway)

   [join market](https://gist.github.com/AdamISZ/4551b947789d3216bacfcb7af25e029e)

2. Pay To EndPoint \(P2EP\)

   [p2ep blockstream](https://blockstream.com/2018/08/08/en-improving-privacy-using-pay-to-endpoint/) “The basic premise of P2EP is that both Sender and Receiver contribute inputs to a transaction via interactions coordinated by an endpoint the Receiver presents using a BIP 21 compliant URI.”

   Steps:

   1. Receiver generates a BIP 21 formatted URI with an additional parameter that specifies their P2EP endpoint.
   2. The Sender initiates interaction with the Receiver by confirming that the endpoint provided is available. If not, the transaction is broadcast normally, paying to the Receiver’s BIP 21 regular Bitcoin address. If the Receiver’s endpoint is available, the Sender provides a signed transaction to the Receiver as proof of UTXO ownership.
   3. The Receiver then sends a number of transactions to the Sender for them to sign. Out of these transactions, only one includes a UTXO that is actually the owned by the Receiver, the rest can be selected from the pool of spendable UTXOs.
   4. Receiver obtains a signed transaction that corresponds to their UTXO they can sign and broadcast the transaction, which will now contain inputs from both the Sender and the Receiver.

   Example: If Alice wants to pay Bob 1 BTC:

   1. Alice inputs 3 BTC to a transaction.
   2. Bob inputs 5 BTC to the same transaction.
   3. Alice receives 2 BTC \(as her change\).
   4. Bob receives 6 BTC \(as his change, plus the 1 BTC payment from Alice\).

   Disadvantages: Receiver and Sender must be online. Interactive. More Cons/Pros listed in blogpost.

### BIP-79 <a id="82e8fb6d-35ec-422d-b78e-1023153f11dc"></a>

[link to bip](https://github.com/bitcoin/bips/blob/master/bip-0079.mediawiki)

### CoinSwaps <a id="f10d2921-5057-43e0-b799-afee706cf651"></a>

[maxwell on coinswaps](https://bitcointalk.org/index.php?topic=321228.0)[waxwing on coinswaps](https://joinmarket.me/blog/blog/coinswaps/)

“We can use a cryptographic commitment scheme to create atomicity that binds two, independent Bitcoin transactions”

Make a random x, hash it. Make a p2sh output that is spendable with proving hash\(x\) is hash in scriptpubkey and pubkey owns output.

Other party can see x and then solve for their p2sh with their pubkey.

[great explainer on cross-chain swaps](https://github.com/AdamISZ/CoinSwapCS/issues/25#issuecomment-311281096)

problem here is that x is revealed and a connection exists between both parties.

HTLCs with presigned transactions can help avoid revealing x.[htlcs wiki](https://en.bitcoin.it/wiki/Hash_Time_Locked_Contracts)

“An advantage of Coinswap over Coinjoin is a potentially bigger anonymity set \(a lot more could be said\)”

[visual guide](https://github.com/AdamISZ/CoinSwapCS/blob/master/docs/coinswap_new.pdf)

[implementation](https://github.com/AdamISZ/CoinSwapCS)

[new coinswap implementation](https://gist.github.com/chris-belcher/9144bd57a91c194e332fb5ca371d0964#design-for-a-coinswap-implementation-for-massively-improving-bitcoin-privacy-and-fungibility)

### TumbleBit <a id="84601db0-0345-4c34-991b-d9027dab506c"></a>

[waxwing on tumblebit](https://joinmarket.me/blog/blog/tumblebit-for-the-tumble-curious/)[original paper](https://eprint.iacr.org/2016/575)

“A blind signature is allows a central authority to sign data which is hidden from them”

“Chaumian cash” is a central mint authorised to blind-sign transfers of this cash

" At a very high level, it’s using commitments - I promise to have X data, by passing over a hashed or encrypted version, but I’m not yet giving it to you - and interactivity - two-way messaging, in particular allowing commitments to occur in both directions."

[blind signatures](https://en.wikipedia.org/wiki/Blind_signature)

### SNICKER <a id="f4bdd9da-c8fe-437c-be88-09db193dc723"></a>

[link to bip](https://gist.github.com/AdamISZ/2c13fb5819bd469ca318156e2cf25d79)

SNICKER \(Simple Non-Interactive Coinjoin with Keys for Encryption Reused\)

allowing the creation of a two party coinjoin without any synchronisation or interaction between the participants.

### PaySwap <a id="8b3926e5-832f-4773-b734-a89bf3a7cf5c"></a>

[dev mailing list](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-January/017595.html)

### More Cryptography <a id="abd2ea2c-909a-43f6-808f-54756c51741f"></a>

1. Adaptor Signatures

   [explainer using atomic swaps](https://github.com/ElementsProject/scriptless-scripts/blob/master/md/atomic-swap.md) “An”adaptor signature" is a not a full, valid signature on a message with your key, but functions as a kind of “promise” that a signature you agree to publish will reveal a secret, or equivalently, allows creation of a valid signature on your key for anyone possessing that secret."

2. Schnorr

   [waxwing on schnorr sigs](https://joinmarket.me/blog/blog/liars-cheats-scammers-and-the-schnorr-signature/)[scriptless scripts and schnorr](https://joinmarket.me/blog/blog/flipping-the-scriptless-script-on-schnorr/)

   [multiparty schnorr coinshuffle](https://joinmarket.me/blog/blog/multiparty-s6/)

3. Ring Signatures

   [waxwing on ring sigs](https://joinmarket.me/blog/blog/ring-sig)

### Chain Analysis <a id="e167a3a9-c683-4853-8ae9-3fe3697b240e"></a>

Peel chains are strings of transactions commonly used for money laundering, in which entities send funds through several wallets in quick succession, usually breaking off small amounts to cash out at each step and sending the majority on to the next wallet.

