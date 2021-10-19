# Bitcoin Core

_Notes on Bitcoin Core architecture and development._

[Bitcoin Core daily coverage reports](https://marcofalke.github.io/btc_cov/)

### Debugging <a id="506a6702-7348-47d0-89dc-0115a6fbc885"></a>

[debug wiki](https://gist.github.com/fjahr/2cd23ad743a2ddfd4eed957274beca0f)

LogPrintf\(""\) cat debug.log \| grep @@@

lldb src/bitcoind

unit tests in src/test/ using BOOST lib test framework.

Run just one test file: src/test/test\_bitcoin –log\_level=all –run\_test=getarg\_tests Run just one test: src/test/test\_bitcoin –log\_level=all –run\_test=\*/the\_one\_test

Logging from unit tests… BOOST\_TEST\_MESSAGE\(“@@@”\);

functional tests in test/functional using python –loglevel=debug self.log.debug\(“bar”\)

Use –tracerpc to see the log outputs from the RPCs of the different nodes running in the functional test in std::out.

[on tests](https://github.com/bitcoin/bitcoin/blob/master/test/README.md)[on unit tests](https://github.com/bitcoin/bitcoin/blob/master/src/test/README.md)[on functional tests](https://github.com/bitcoin/bitcoin/blob/master/test/functional/README.md)

[core review tools](https://github.com/fanquake/core-review)

### Architecture <a id="24df961f-e563-434c-860d-89457aa81cde"></a>

[overview of arch](https://jameso.be/dev++2018/#1)

### Wallet <a id="50d67092-4617-41fe-9f22-a22fedfae291"></a>

[wallet dev presentation by John Newbery](https://residency.chaincode.com/presentations/bitcoin/Wallet_Development.pdf) CPubKey - a public key, used to verify signatures. A point on the secp256k1 curve. CKey - an encapsulated private key. Used to sign data. CKeyID - a key identifier, which is the RIPEMD160\(SHA256\(pubkey\)\) CTxDestination - a txout script template with a specific destination. Stored as a varint variable

* CNoDestination: no destination set
* CKeyID: P2PKH
* CScriptID: P2SH
* WitnessV0ScriptHash
* WitnessV0KeyHash
* WitnessUnknown

Wallet component is intialized through the WalletInitInterface. For builds with wallet, the interface is overrridden in src/wallet/init.cpp

For –disable-wallet, there is DummyWalletInit

initiation interface methods are called during node initialization

During loading… WalletInit::Construct\(\) adds a client interface to the wallet. Node then tells wallet to load/start/stop/etc through the ChainClient interface in src/interfaces/wallet.cpp Most methods in that interface call through to functions in src/wallet/load.cpp

Node &lt;&gt; Wallet Interface Node holds a WAlletImpl interface to call functions on the wallet. Wallet holds a ChainImpl interface to call functions on the node. Notifications handler Node notifies the wallet about new transactions and blocks through the CValidationInterface

Identifying Transactions When a transaction is added to the mempool or block is “connected”, the wallet is notified through CValidationInterface. SyncTransaction\(\) … calls AddToWalletIfInvolvingMe\(\) IsMine\(\) : takes the scriptPubKey, interprets it as a Destination type, and then checks whether we have the key\(s\) to watch/spend.

Generate Keys Originally a collection of unrelated private keys. Keypools introduced in 2010 by Satoshi. Cache 100 private keys. When a new key is needed, draw it from keypool and refresh. HD wallets introduced to Bitcoin Core in 2016. Keypool essentially became an address lock-ahead pool. It is used to implement a ‘gap limit’.

Constructing Transactions sendtoaddress sendtomany {create,fund,sign,send}rawtransaction The address is decoded into a CDestination. Other parameters can be added for finer control \(RBF, fees, etc\). Wallet creates the transaction in CreateTransaction\(\).

Coin Selection By default, coin selection is automatic. Logic starts in CWallet:SelectCoins\(\). By preference, we choose coins with more confirmations. Manual coin selection \(coin control\) is possible in CCoinControl.

Signing Inputs Last step in CreateTransaction\(\) CWallet is an implementation of SigningProvider interface. Signing logic for the SigningProvider is all in src/script/sign.cpp.

Sending Transactions Wallet saves and broadcats the wallet in CommitTransaction\(\) submitToMemoryPool\(\), relayTransaction\(\)

