# Simulate Transactions on the Bitcoin Regtest Network

## ⚙️ Configuring the Bitcoin Environment

In `~/Library/Application\ Support/Bitcoin/bitcoin.conf`, add the following settings:

```bash
rpcuser=nishant
rpcpassword=nishant
regtest=1
daemon=1
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
zmqpubhashtx=tcp://127.0.0.1:28332
zmqpubhashblock=tcp://127.0.0.1:28332
txindex=1
fallbackfee=0.00072
mintxfee=0.0001
[signet]
rpcport=18443
```

**NOTE**: Bash commands are starting with $.

### About ZeroMQ (ZMQ)
ZeroMQ, or ZMQ, https://zeromq.org/, provides a method for bitcoind to share block and transaction information.

### Specify Blockchain Data Directory
Tell Bitcoin where to put blockchain data:
`datadir=/path/to/my/blockchain/data/.bitcoin/data`

### Enable RPC
Remote Procedure Call, or RPC, https://en.wikipedia.org/wiki/Remote_procedure_call, is the way in which bitcoind interacts with other programs. To enable this interaction you'll need to set a password and turn on the RPC server.

Set the RPC password:
`rpcauth=mysupersecurepassword`
 

Turn on the RPC server:
`server=1`

## Commands:
1. Starting the Bitcoin Core:
	`bitcoind`

2. Creating a wallet on regtest network
- Create a wallet:
	`bitcoin-cli createwallet "test-wallet-1"`

- Load an existing wallet:
	`bitcoin-cli loadwallet "test-wallet-1"`

- List all wallets:
	`bitcoin-cli listwallets`

- Backup Wallet(Before you test anything crucial, make sure to back up your wallet):
	`bitcoin-cli backupwallet <filename>`

- Encrypt Wallet(Always encrypt your wallet to protect your private keys):
	`bitcoin-cli encryptwallet <passphrase>`

3. After Encrytpint the wallet
- Unlock the wallet for 60 seconds:
	`bitcoin-cli -rpcwallet=test-wallet-1 walletpassphrase "test-wallet-1" 60`

- Rescan the Blockchain (for new transactions):
	`bitcoin-cli -rpcwallet=test-wallet-1 rescanblockchain`


4. Now we can mine some blocks.(Generate blocks, equivalent to RPC getnewaddress followed by RPC generatetoaddress. )
	`bitcoin-cli -generate 10`

5. You must mine 101 blocks before your bitcoind node has matured bitcoin to transfer to another wallet after doing that Now you should see a positive balance:
	`bitcoin-cli getwalletinfo`

6. Retrieve Blockchain Information:
	`bitcoin-cli -getinfo`

7. Check the blocks are mined
	`bitcoin-cli getblockcount`

8. Wallet balance:
	`bitcoin-cli getwalletinfo`

9. Get information about the current state of the blockchain
	`bitcoin-cli getblockchaininfo`

10. Create a new Bitcoin address to receive payments
	`bitcoin-cli getnewaddress`

11. To list all transactions in the wallet: 
	`bitcoin-cli listtransactions`

12. Checking Network and Connection Status
- Check Network Info: `bitcoin-cli getnetworkinfo`
- Check Connection Status(To see the list of connected peers): `bitcoin-cli getpeerinfo`

13. To retrieve information about a specific transaction, use the getrawtransaction command. You will need the transaction ID (txid), which is returned when you send a transaction or generate blocks.
	`bitcoin-cli getrawtransaction <txid> true`

14. Use the listaddressgroupings command to list all addresses in the wallet that currently hold a balance(Bitcoin wallets can have multiple addresses associated with them for privacy and organizational purposes. Each address is independent, but they are all linked to the same wallet and can hold funds.): 
	`bitcoin-cli listaddressgroupings`

15. You can disable fee estimation completely and manually specify a fee rate using the -paytxfee parameter when sending a transaction:
	`bitcoin-cli -rpcwallet=test-wallet-1 settxfee 0.1`

16. You can send Bitcoin from one address to another within the same wallet also:
	`bitcoin-cli sendtoaddress bcrt1q2tgzp5yffpdxlhezg4rt2fycck4mht0kdanh4f 100`

17. The gettransaction command returns detailed information about a transaction, including its confirmation status:
	`bitcoin-cli gettransaction d6669732e71a5752f78e851307ebb4ee4f4078daa80abd398430d69b2dcfd10d`
this <txid> is we got from `sendtoaddress`
("confirmations": 0) in the above command output means it is not yet confirmed

Check for Mempool Status: To see if your transaction is in the mempool (unconfirmed transactions), you can use:
	`bitcoin-cli getrawmempool`
This will list all unconfirmed transactions. Your transaction d6669732e71a5752f78e851307ebb4ee4f4078daa80abd398430d69b2dcfd10d should appear here if it’s not yet confirmed.

Since regtest is a private network where you control the block creation, you can mine blocks to confirm transactions. You can use the generate command to mine a block. By default, Bitcoin Core mines a block every time you call generate in regtest mode.

To mine a block and confirm your transaction, run:
	`bitcoin-cli -generate 1`

18. After mining the block, you can check the status of the transaction using:
	`bitcoin-cli gettransaction d6669732e71a5752f78e851307ebb4ee4f4078daa80abd398430d69b2dcfd10d`
The confirmations field should show 1 (or more, depending on how many blocks you've mined).
6 Confirmations: This is generally considered the standard for fully confirmed transactions on the Bitcoin.(The reason 6 is typically the standard is that, with a 51% attack (where an attacker controls more than half of the network's mining power), it becomes increasingly difficult to reverse a transaction as more blocks are added on top of it. After 6 blocks, the probability of an attack succeeding and reversing your transaction becomes very low.)

## ⚙️ Configuring our two Bitcoin nodes
### Configurations
```bash
export BITCOIN1_DIR=/Users/nishantbansal/Desktop/Open-Source/SoB/Send-Recieve\ BTC/bitcoin-node-1
export BITCOIN2_DIR=/Users/nishantbansal/Desktop/Open-Source/SoB/Send-Recieve\ BTC/bitcoin-node-2
```

In `$BITCOIN1_DIR/bitcoin.conf` add the below settings:
```bash
# bitcoin.conf for Node 1
regtest=1
daemon=1
server=1
debug=1
rpcuser=bitcoinuser1
rpcpassword=bitcoinpassword1
rpcallowip=127.0.0.1
[regtest]
rpcport=18443
port=8333
```

In `$BITCOIN12_DIR/bitcoin.conf` add the below settings:
```bash
# bitcoin.conf for Node 2
regtest=1
daemon=1
server=1
rpcuser=bitcoinuser2
rpcpassword=bitcoinpassword2
rpcallowip=127.0.0.1
[regtest]
connect=127.0.0.1:8333
rpcport=18444
port=8334
```

**NOTE:** We need to use another port and another data dir.

**Summary:**
Create 2 folder, with different bitcoin.conf(having different port and tpcport), node-2 should have connect in their bitcoin.conf file otherwise the node-2 will shutdown immidiately on start(because Normally, Bitcoin nodes can discover peers on the network using DNS seeds or other nodes they connect to. However, since you're using regtest, the node does not try to discover external peers and won't automatically find bitcoin-node-1 unless you explicitly tell it to connect to that node. That's why, in some cases, adding the connect option helps ensure the nodes are connected.)

## Running Bitcoin regtest network
1. Start both bitcoin nodes: `bitcoind -datadir=$BITCOIN1_DIR` and `bitcoind -datadir=$BITCOIN2_DIR`
2.[Optional] Connect to our second node: `bitcoin-cli -datadir=$BITCOIN1_DIR addnode "127.0.0.1:8334" add`
To keep the connection after a restart, add the following to your bitcoin.conf: `addnode=127.0.0.1:8334`
3.[Optional] Check if your nodes are connected: `bitcoin-cli -datadir=$BITCOIN1_DIR getaddednodeinfo`(provides information about the nodes that have been added to your Bitcoin client as peer nodes. These are the nodes that your client is aware of and may connect to in order to propagate transactions, blocks, and synchronize data on the Bitcoin network.)
OUTPUT:
```bash
[
  {
    "addednode": "127.0.0.1:8334",
    "connected": false,
    "addresses": [
    ]
  }
]
```
4. First Let's create/load wallet in both the nodes
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR createwallet "btc-wallet-1"
{
  "name": "btc-wallet-1"
}
```

```bash
$ bitcoin-cli -datadir=$BITCOIN2_DIR createwallet "btc-wallet-2"
{
  "name": "btc-wallet-2"
}
```
5. Now let's generate/min some blocks to get BTC for transactions:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR -generate 101
```
verify the funds:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR getwalletinfo
{
  "walletname": "btc-wallet-1",
  "walletversion": 169900,
  "format": "sqlite",
  "balance": 8800.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 2450.00000000,
  "txcount": 303,
  "keypoolsize": 4000,
  "keypoolsize_hd_internal": 4000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true,
  "external_signer": false,
  "blank": false,
  "birthtime": 1735896385,
  "lastprocessedblock": {
    "hash": "2e690e6bd49d785d9a14b27e3afdf81e14a3f8699fe4e53738ba624db03208d7",
    "height": 303
  }
}
```
check the balance:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR getbalance
8800.00000000
```
6. Create a transaction
Now, create a transaction from one node to the other (from bitcoin1 to bitcoin2).

On bitcoin1, get a list of unspent outputs (UTXOs) available to send:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR listunspent
[
	{
    "txid": "d73d8d4e739109934d8bc727cbfe87fc1b872d6bfae797407f8a2913daff6ced",
    "vout": 0,
    "address": "bcrt1qd5txsxl79vf8zhgjelwep3488fcf3zsg9pahv3",
    "label": "",
    "scriptPubKey": "00146d16681bfe2b12715d12cfdd90c6a73a70988a08",
    "amount": 50.00000000,
    "confirmations": 302,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([eaaa6a70/84h/1h/0h/0/0]03dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f)#2h7vmjuu",
    "parent_descs": [
      "wpkh(tpubD6NzVbkrYhZ4XGFzG9EW95Xm6iiCDumhkKi13kGt6qXsz9CNLhLDQfv7wGtLhUoYemWAusAS2DwKL1vXc8M8r2w1vwyVc2QjHDHEJXhJxX8/84h/1h/0h/0/*)#gu4tacry"
    ],
    "safe": true
  },
................
]
```
7. Currently, In bitcoind2
```bash
bitcoin-cli -datadir=$BITCOIN2_DIR getbalance
0.00000000
```
and

```bash
bitcoin-cli -datadir=$BITCOIN2_DIR getwalletinfo                                                                                                                          ─╯
{
  "walletname": "btc-wallet-2",
  "walletversion": 169900,
  "format": "sqlite",
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoolsize": 4000,
  "keypoolsize_hd_internal": 4000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true,
  "external_signer": false,
  "blank": false,
  "birthtime": 1735896460,
  "lastprocessedblock": {
    "hash": "2e690e6bd49d785d9a14b27e3afdf81e14a3f8699fe4e53738ba624db03208d7",
    "height": 303
  }
}
```
8. Obtain the address of the second node by running:
```bash
$ bitcoin-cli -datadir=$BITCOIN2_DIR getnewaddress
bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy
```

9. Send/create a transaction to bitcoin-cli -datadir=$BITCOIN2_DIR:
```bash
bitcoin-cli -datadir=$BITCOIN1_DIR sendtoaddress <bitcoin2_address> <amount>
```
Replace <bitcoin2_address> with the address of bitcoin2 and <amount> with the amount of Bitcoin you want to send.
Ex:-
```bash
bitcoin-cli -datadir=$BITCOIN1_DIR sendtoaddress bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy 500
```
**NOTE:**
May return an error like:
```bash
bitcoin-cli -datadir=$BITCOIN1_DIR sendtoaddress bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy 500                                                                         ─╯
error code: -6
error message:
Fee estimation failed. Fallbackfee is disabled. Wait a few blocks or enable -fallbackfee.
```
### 2 ways to fix it is 
1. 
```bash
$ bitcoin-cli -rpcwallet=test-wallet-1 settxfee 0.1
true
```
2. Use -fallbackfee (recommended for testing):

As mentioned earlier, you can specify a fallback fee, which will allow the transaction to go through despite fee estimation not working in regtest mode.

To do this, you can start your Bitcoin node with the -fallbackfee option:

```
bitcoind -datadir=$BITCOIN1_DIR -fallbackfee=0.0001
```
Alternatively, you can add the following to your bitcoin.conf:
```
fallbackfee=0.0001
```
This will set a default fee of 0.0001 BTC per kilobyte, which is a small enough fee to use in regtest.

So, Now after fixing the above issue:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR sendtoaddress bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy 500
d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e
```
10. Check if the transaction was sent successfully
To see the status of your transaction, run this on bitcoin1:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR getrawmempool
[
  "d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e"
]
```
This will show any unconfirmed transactions in the mempool.

11. Check transaction confirmation:
on node1:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR gettransaction d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e
{
  "amount": -500.00000000,
  "fee": -0.08180000,
  "confirmations": 0,
  "trusted": true,
  "txid": "d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e",
  "wtxid": "ab766bb28ccc78f46af3bda076e65d055f0f315c6478f59c77f33b31e52b5503",
  "walletconflicts": [
  ],
  "mempoolconflicts": [
  ],
  "time": 1735897596,
  "timereceived": 1735897596,
  "bip125-replaceable": "yes",
  "details": [
    {
      "address": "bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy",
      "category": "send",
      "amount": -500.00000000,
      "vout": 0,
      "fee": -0.08180000,
      "abandoned": false
    }
  ],
  "hex": "0200000000010bf56dba7c464ebbbac622bcdc15e183c894cee46c98297143d033233131dc400f0000000000fdffffff5146eb67ebb980cf5c2a54b729b47504c1b4f46bf013ec6d4955c74d5fa54e5c0000000000fdffffff7d5ff007a837d9ba1af66ee09d2e8340731a17554eb072f6907135a95e0d93e40000000000fdffffff222ac9a4dfb0422689055deec5044816d79ecb5d127cc67f8932dc91b7b73e5b0000000000fdffffff0d35b37641f22f51db387122e91670a43a0c0bdcd5a7cf7b0ed8196d6cde66140000000000fdffffffdf50968cac951ac7cf5daa7df4548583747d56451af54e7e49dfe0d3528b86f00000000000fdffffffaec380da2b78eba0992b7f64571d2f87fbbf490108a566fde36025043529fe5f0000000000fdffffffb3530f6319eaf3d0c8f5a7ca3398d7dde99d4ed912351ef47e2f97cd8d7b30650000000000fdffffffb20ee473b634af7c69b0b5238d77e09dc215e64e2a3a2d3725805f3fbc263f1b0000000000fdffffff9a66c91fcb94c7c2b331e3e31fb545bf5ab8e753d669fc9b2d0bb68f27eeac680000000000fdffffff0495cc8f8211da83f365bf41faabe981125984504a4ccc390863cef36048313a0000000000fdffffff0200743ba40b000000160014452ff3a1daeed9082bba594183dc84d4f2155d4de027869400000000160014d6a2a45e672c19e51c81e8921f08697287b400410247304402205844f94d674f7c1c11c37639c53cb909a54451aad63880bde9cef62c485af89302203313ac30f7efe173e78bd3654f48ebfae366b3aa7f90cd370ab7b4a6f4e6ab4e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022022b9e75dde3cc65e5183005fe6b5d3394a3cddc907c19c21c97a48ea6d6fdf5e02205d1ffd745f67ca7268aff6a540cea252e3ed326211fd39442a004f13f8eda186012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022031014587c2e52f2cf4ecf063e7c53a5800b540bf0e71c9c9d01d34a6adf979bb02201f5766630de92be9fc562ab7f040ff70074bd6e5621566a0567ccd81a6effe54012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402202a36210bc93759758277b39abc893c6be3a186fe5e44a8ca47de7e02bd191d13022066759acf82867557abcc3187c21a3b0b301edce98dcabcf0c573c2ead166273c012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402203fdff05bc7c7bab3d2264dc2cd735e15f175b03fe7f445b1e8bdb4f5d1641fc302200d1e19043966f2edb045842a6766b691f32658ad8ae209e1112d386619c25da5012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220300a1b3fa31d1305a503947e7add60c49c7fb264e370b0fd203b0855f4f75a7402201f613878ed97d2f6a90a5f48dce37faefda03a57175961fcc4d90f3e1374f000012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220043d79bed91c6c28231b780fe48b7b1d9791ee7f4ff578ad62b628d5c4833cfb022047af553c071a30188ea554fad7754ae636bfa266000e4ad855edc6aeefac75f2012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402201c574511f9dbceff1aa9ae828b05745b3d16ec442d669f8ef1a1fd1099c65d92022007adaf53f81343325a4e043fc963f7cbbf39e8f62c02294ee9adfb27c9c0e77e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022039a98c7c795a5d254ecf18aa328bac0ae8c597e2e729e55fda256a4ccb7816a4022029c33f83264a157576f936a8416d02a71c6184b4e4216835309a5e5502ffec96012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402201c1ac40928961bbba3258e93e5564039f1cd73426213b9d7cf9ed31cd36eca17022055aa49ce3d196038bd3f146062d4ca51f0581c8bc14f1430874807ab9cbc1584012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220254df624f8e75ed8d71cdfb865f6a7a9afff93c344b960b58ed6b1f394dd9bbd022020342aeaa3a51af2ce0d6c11e1da0215aff1cca98c8ef55f6756a2eae40b5293012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca1b010000",
  "lastprocessedblock": {
    "hash": "2e690e6bd49d785d9a14b27e3afdf81e14a3f8699fe4e53738ba624db03208d7",
    "height": 303
  }
}
```

one node2:
```bash
bitcoin-cli -datadir=$BITCOIN2_DIR gettransaction d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e
{
  "amount": 500.00000000,
  "confirmations": 0,
  "trusted": false,
  "txid": "d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e",
  "wtxid": "ab766bb28ccc78f46af3bda076e65d055f0f315c6478f59c77f33b31e52b5503",
  "walletconflicts": [
  ],
  "mempoolconflicts": [
  ],
  "time": 1735897598,
  "timereceived": 1735897598,
  "bip125-replaceable": "yes",
  "details": [
    {
      "address": "bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy",
      "parent_descs": [
        "wpkh(tpubD6NzVbkrYhZ4WLfo85HqvASxd44Ug2Biiqx2gKTSoccVnCv4fae5E6svuzrUCgVq2pkfbxe5jANLf3rjyZGecDaE7379CtsSYMvcqBXmHVM/84h/1h/0h/0/*)#n5ty7csh"
      ],
      "category": "receive",
      "amount": 500.00000000,
      "label": "",
      "vout": 0,
      "abandoned": false
    }
  ],
  "hex": "0200000000010bf56dba7c464ebbbac622bcdc15e183c894cee46c98297143d033233131dc400f0000000000fdffffff5146eb67ebb980cf5c2a54b729b47504c1b4f46bf013ec6d4955c74d5fa54e5c0000000000fdffffff7d5ff007a837d9ba1af66ee09d2e8340731a17554eb072f6907135a95e0d93e40000000000fdffffff222ac9a4dfb0422689055deec5044816d79ecb5d127cc67f8932dc91b7b73e5b0000000000fdffffff0d35b37641f22f51db387122e91670a43a0c0bdcd5a7cf7b0ed8196d6cde66140000000000fdffffffdf50968cac951ac7cf5daa7df4548583747d56451af54e7e49dfe0d3528b86f00000000000fdffffffaec380da2b78eba0992b7f64571d2f87fbbf490108a566fde36025043529fe5f0000000000fdffffffb3530f6319eaf3d0c8f5a7ca3398d7dde99d4ed912351ef47e2f97cd8d7b30650000000000fdffffffb20ee473b634af7c69b0b5238d77e09dc215e64e2a3a2d3725805f3fbc263f1b0000000000fdffffff9a66c91fcb94c7c2b331e3e31fb545bf5ab8e753d669fc9b2d0bb68f27eeac680000000000fdffffff0495cc8f8211da83f365bf41faabe981125984504a4ccc390863cef36048313a0000000000fdffffff0200743ba40b000000160014452ff3a1daeed9082bba594183dc84d4f2155d4de027869400000000160014d6a2a45e672c19e51c81e8921f08697287b400410247304402205844f94d674f7c1c11c37639c53cb909a54451aad63880bde9cef62c485af89302203313ac30f7efe173e78bd3654f48ebfae366b3aa7f90cd370ab7b4a6f4e6ab4e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022022b9e75dde3cc65e5183005fe6b5d3394a3cddc907c19c21c97a48ea6d6fdf5e02205d1ffd745f67ca7268aff6a540cea252e3ed326211fd39442a004f13f8eda186012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022031014587c2e52f2cf4ecf063e7c53a5800b540bf0e71c9c9d01d34a6adf979bb02201f5766630de92be9fc562ab7f040ff70074bd6e5621566a0567ccd81a6effe54012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402202a36210bc93759758277b39abc893c6be3a186fe5e44a8ca47de7e02bd191d13022066759acf82867557abcc3187c21a3b0b301edce98dcabcf0c573c2ead166273c012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402203fdff05bc7c7bab3d2264dc2cd735e15f175b03fe7f445b1e8bdb4f5d1641fc302200d1e19043966f2edb045842a6766b691f32658ad8ae209e1112d386619c25da5012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220300a1b3fa31d1305a503947e7add60c49c7fb264e370b0fd203b0855f4f75a7402201f613878ed97d2f6a90a5f48dce37faefda03a57175961fcc4d90f3e1374f000012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220043d79bed91c6c28231b780fe48b7b1d9791ee7f4ff578ad62b628d5c4833cfb022047af553c071a30188ea554fad7754ae636bfa266000e4ad855edc6aeefac75f2012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402201c574511f9dbceff1aa9ae828b05745b3d16ec442d669f8ef1a1fd1099c65d92022007adaf53f81343325a4e043fc963f7cbbf39e8f62c02294ee9adfb27c9c0e77e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022039a98c7c795a5d254ecf18aa328bac0ae8c597e2e729e55fda256a4ccb7816a4022029c33f83264a157576f936a8416d02a71c6184b4e4216835309a5e5502ffec96012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402201c1ac40928961bbba3258e93e5564039f1cd73426213b9d7cf9ed31cd36eca17022055aa49ce3d196038bd3f146062d4ca51f0581c8bc14f1430874807ab9cbc1584012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220254df624f8e75ed8d71cdfb865f6a7a9afff93c344b960b58ed6b1f394dd9bbd022020342aeaa3a51af2ce0d6c11e1da0215aff1cca98c8ef55f6756a2eae40b5293012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca1b010000",
  "lastprocessedblock": {
    "hash": "2e690e6bd49d785d9a14b27e3afdf81e14a3f8699fe4e53738ba624db03208d7",
    "height": 303
  }
}
```
Similarly, on bitcoin-2, you can check if the transaction was received:
```bash
$ bitcoin-cli -datadir=$BITCOIN2_DIR getreceivedbyaddress bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy
0.00000000
```
Since it is not confirmed

12. Let's confirm the transaction by mining some blocks:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR -generate 6
{
  "address": "bcrt1qgnnc9x7vsga2z9lj43gmd2pcglfdrnl06fsq9w",
  "blocks": [
    "55349940c769fed847d36fe582723dde82a7d1ebfeedf97caa7cc790270ab10d",
    "2fdb7ae26761618509fb9be817130759829db409febdcf99fe641b7b1eff1bf1",
    "38960c98109a04eef15f45387b2909e1eeeccc2b5ba3ca92dc37c4de337021f6",
    "2a036a9a24ec4aa07ea62c0e0b9524879383cd08ca8bfe8f2591dacd5b0b75ce",
    "3d779c8eebb4eb0c9e35dec0976cdb30902dd6374878a2d27d9719a79023f4ab",
    "0de0a444c7ce22a9ec138c822a45c7479239c2b57d7fae5b479df1181723431c"
  ]
}
```
13. Now we can see the transaction is recieved on bitcoin-2:
```bash
bitcoin-cli -datadir=$BITCOIN2_DIR getreceivedbyaddress bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy
500.00000000
```
```bash
$ bitcoin-cli -datadir=$BITCOIN2_DIR gettransaction d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e
{
  "amount": 500.00000000,
  "confirmations": 6,
  "blockhash": "55349940c769fed847d36fe582723dde82a7d1ebfeedf97caa7cc790270ab10d",
  "blockheight": 304,
  "blockindex": 1,
  "blocktime": 1735897977,
  "txid": "d80261d17068e687339159f9686b735fff663a52f11dac1a8c522e5e0efc046e",
  "wtxid": "ab766bb28ccc78f46af3bda076e65d055f0f315c6478f59c77f33b31e52b5503",
  "walletconflicts": [
  ],
  "mempoolconflicts": [
  ],
  "time": 1735897598,
  "timereceived": 1735897598,
  "bip125-replaceable": "no",
  "details": [
    {
      "address": "bcrt1qg5hl8gw6amvss2a6t9qc8hyy6nep2h2dfdeazy",
      "parent_descs": [
        "wpkh(tpubD6NzVbkrYhZ4WLfo85HqvASxd44Ug2Biiqx2gKTSoccVnCv4fae5E6svuzrUCgVq2pkfbxe5jANLf3rjyZGecDaE7379CtsSYMvcqBXmHVM/84h/1h/0h/0/*)#n5ty7csh"
      ],
      "category": "receive",
      "amount": 500.00000000,
      "label": "",
      "vout": 0,
      "abandoned": false
    }
  ],
  "hex": "0200000000010bf56dba7c464ebbbac622bcdc15e183c894cee46c98297143d033233131dc400f0000000000fdffffff5146eb67ebb980cf5c2a54b729b47504c1b4f46bf013ec6d4955c74d5fa54e5c0000000000fdffffff7d5ff007a837d9ba1af66ee09d2e8340731a17554eb072f6907135a95e0d93e40000000000fdffffff222ac9a4dfb0422689055deec5044816d79ecb5d127cc67f8932dc91b7b73e5b0000000000fdffffff0d35b37641f22f51db387122e91670a43a0c0bdcd5a7cf7b0ed8196d6cde66140000000000fdffffffdf50968cac951ac7cf5daa7df4548583747d56451af54e7e49dfe0d3528b86f00000000000fdffffffaec380da2b78eba0992b7f64571d2f87fbbf490108a566fde36025043529fe5f0000000000fdffffffb3530f6319eaf3d0c8f5a7ca3398d7dde99d4ed912351ef47e2f97cd8d7b30650000000000fdffffffb20ee473b634af7c69b0b5238d77e09dc215e64e2a3a2d3725805f3fbc263f1b0000000000fdffffff9a66c91fcb94c7c2b331e3e31fb545bf5ab8e753d669fc9b2d0bb68f27eeac680000000000fdffffff0495cc8f8211da83f365bf41faabe981125984504a4ccc390863cef36048313a0000000000fdffffff0200743ba40b000000160014452ff3a1daeed9082bba594183dc84d4f2155d4de027869400000000160014d6a2a45e672c19e51c81e8921f08697287b400410247304402205844f94d674f7c1c11c37639c53cb909a54451aad63880bde9cef62c485af89302203313ac30f7efe173e78bd3654f48ebfae366b3aa7f90cd370ab7b4a6f4e6ab4e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022022b9e75dde3cc65e5183005fe6b5d3394a3cddc907c19c21c97a48ea6d6fdf5e02205d1ffd745f67ca7268aff6a540cea252e3ed326211fd39442a004f13f8eda186012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022031014587c2e52f2cf4ecf063e7c53a5800b540bf0e71c9c9d01d34a6adf979bb02201f5766630de92be9fc562ab7f040ff70074bd6e5621566a0567ccd81a6effe54012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402202a36210bc93759758277b39abc893c6be3a186fe5e44a8ca47de7e02bd191d13022066759acf82867557abcc3187c21a3b0b301edce98dcabcf0c573c2ead166273c012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402203fdff05bc7c7bab3d2264dc2cd735e15f175b03fe7f445b1e8bdb4f5d1641fc302200d1e19043966f2edb045842a6766b691f32658ad8ae209e1112d386619c25da5012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220300a1b3fa31d1305a503947e7add60c49c7fb264e370b0fd203b0855f4f75a7402201f613878ed97d2f6a90a5f48dce37faefda03a57175961fcc4d90f3e1374f000012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220043d79bed91c6c28231b780fe48b7b1d9791ee7f4ff578ad62b628d5c4833cfb022047af553c071a30188ea554fad7754ae636bfa266000e4ad855edc6aeefac75f2012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f0247304402201c574511f9dbceff1aa9ae828b05745b3d16ec442d669f8ef1a1fd1099c65d92022007adaf53f81343325a4e043fc963f7cbbf39e8f62c02294ee9adfb27c9c0e77e012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f02473044022039a98c7c795a5d254ecf18aa328bac0ae8c597e2e729e55fda256a4ccb7816a4022029c33f83264a157576f936a8416d02a71c6184b4e4216835309a5e5502ffec96012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca0247304402201c1ac40928961bbba3258e93e5564039f1cd73426213b9d7cf9ed31cd36eca17022055aa49ce3d196038bd3f146062d4ca51f0581c8bc14f1430874807ab9cbc1584012103dc5833bd49e3cc251ead259e32297fc1145ca0072e79cac12cb76819c619368f024730440220254df624f8e75ed8d71cdfb865f6a7a9afff93c344b960b58ed6b1f394dd9bbd022020342aeaa3a51af2ce0d6c11e1da0215aff1cca98c8ef55f6756a2eae40b5293012102c81f150ba036dd49d32799d145d6054728ef2eaf097e2abc58b7b75832c66cca1b010000",
  "lastprocessedblock": {
    "hash": "0de0a444c7ce22a9ec138c822a45c7479239c2b57d7fae5b479df1181723431c",
    "height": 309
  }
}
```

14. Finally, make sure the two nodes are synchronized and connected by checking the block count and syncing status.

On bitcoin-1:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR getblockcount
309
```
Similarly, check on bitcoin-2.

Both nodes should have the same block height once the transaction is confirmed

## Extras
Source:
- https://gist.github.com/System-Glitch/cb4e87bf1ae3fec9925725bb3ebe223a

## Additional
The output from the getpeerinfo command shows the current state of the peer connections for both Bitcoin nodes. Here’s a breakdown of the key fields and what they indicate about your nodes’ connection:

For Node 1 (BITCOIN1_DIR):
```json
{
  "id": 0,
  "addr": "127.0.0.1:55761",
  "addrbind": "127.0.0.1:8333",
  "network": "not_publicly_routable",
  "services": "0000000000000c09",
  "servicesnames": ["NETWORK", "WITNESS", "NETWORK_LIMITED", "P2P_V2"],
  "relaytxes": true,
  "lastsend": 1735898353,
  "lastrecv": 1735898353,
  "last_transaction": 0,
  "last_block": 0,
  "bytessent": 94682,
  "bytesrecv": 2423,
  "conntime": 1735895832,
  "timeoffset": 0,
  "pingtime": 0.000195,
  "minping": 9.6e-05,
  "version": 70016,
  "subver": "/Satoshi:28.0.0/",
  "inbound": true,
  "bip152_hb_to": false,
  "bip152_hb_from": true,
  "startingheight": 0,
  "presynced_headers": -1,
  "synced_headers": -1,
  "synced_blocks": -1,
  "inflight": [],
  "addr_relay_enabled": true,
  "addr_processed": 0,
  "addr_rate_limited": 0,
  "minfeefilter": 0.00001000,
  "bytessent_per_msg": {
    "addrv2": 49,
    "block": 1345,
    "cmpctblock": 85297,
    "feefilter": 58,
    "getheaders": 90,
    "headers": 449,
    "inv": 116,
    "ping": 638,
    "pong": 638,
    "sendaddrv2": 33,
    "sendcmpct": 30,
    "tx": 1723,
    "verack": 33,
    "version": 135,
    "wtxidrelay": 33
  },
  "bytesrecv_per_msg": {
    "feefilter": 58,
    "getaddr": 33,
    "getdata": 260,
    "getheaders": 180,
    "headers": 22,
    "ping": 638,
    "pong": 638,
    "sendaddrv2": 33,
    "sendcmpct": 60,
    "sendheaders": 33,
    "verack": 33,
    "version": 135,
    "wtxidrelay": 33
  },
  "connection_type": "inbound",
  "transport_protocol_type": "v2",
  "session_id": "bc1615425f8b436243165b082cdab39adc340a4bebc2aede945dce8f5c80a2b3"
}
```
addr: The peer's address (127.0.0.1:55761), which is a local connection.
addrbind: The Bitcoin node's bind address (127.0.0.1:8333), which is the listening port for incoming connections.
network: "not_publicly_routable" indicates this peer is not publicly routable and is part of a local network.
servicesnames: Lists the services supported by the peer, such as P2P, Witness support, and other Bitcoin network services.
inbound: true indicates this connection was initiated by the other peer (in this case, Node 2).
pingtime: Shows the latency of the connection (low ping time is good).
synced_headers and synced_blocks: Both are -1, which suggests that synchronization might be in the process but not yet completed.
connection_type: inbound, meaning this is an incoming connection to this node.
For Node 2 (BITCOIN2_DIR):
```json
{
  "id": 0,
  "addr": "127.0.0.1:8333",
  "addrbind": "127.0.0.1:55761",
  "network": "not_publicly_routable",
  "services": "0000000000000c09",
  "servicesnames": ["NETWORK", "WITNESS", "NETWORK_LIMITED", "P2P_V2"],
  "relaytxes": true,
  "lastsend": 1735898353,
  "lastrecv": 1735898353,
  "last_transaction": 1735897598,
  "last_block": 1735897977,
  "bytessent": 2423,
  "bytesrecv": 94682,
  "conntime": 1735895832,
  "timeoffset": 0,
  "pingtime": 0.00078,
  "minping": 0.000163,
  "version": 70016,
  "subver": "/Satoshi:28.0.0/",
  "inbound": false,
  "bip152_hb_to": true,
  "bip152_hb_from": false,
  "startingheight": 0,
  "presynced_headers": -1,
  "synced_headers": 309,
  "synced_blocks": 309,
  "inflight": [],
  "addr_relay_enabled": true,
  "addr_processed": 1,
  "addr_rate_limited": 0,
  "permissions": [],
  "minfeefilter": 0.00001000,
  "bytessent_per_msg": {
    "feefilter": 58,
    "getaddr": 33,
    "getdata": 260,
    "getheaders": 180,
    "headers": 22,
    "ping": 638,
    "pong": 638,
    "sendaddrv2": 33,
    "sendcmpct": 60,
    "sendheaders": 33,
    "verack": 33,
    "version": 135,
    "wtxidrelay": 33
  },
  "bytesrecv_per_msg": {
    "addrv2": 49,
    "block": 1345,
    "cmpctblock": 85297,
    "feefilter": 58,
    "getheaders": 90,
    "headers": 449,
    "ping": 638,
    "pong": 638,
    "sendaddrv2": 33,
    "sendcmpct": 30,
    "sendheaders": 33,
    "verack": 33,
    "version": 135,
    "wtxidrelay": 33
  },
  "connection_type": "manual",
  "transport_protocol_type": "v2",
  "session_id": "bc1615425f8b436243165b082cdab39adc340a4bebc2aede945dce8f5c80a2b3"
}
```
addr: The peer's address (127.0.0.1:8333).
addrbind: The Bitcoin node's bind address (127.0.0.1:55761).
inbound: false means this node initiated the connection, so it is an outbound connection from Node 2 to Node 1.
synced_headers and synced_blocks: The peer has synced 309 headers and blocks, indicating that it has synced blocks up to this height.
connection_type: manual indicates that the connection was manually set up via addnode (from Node 1 to Node 2).
What Does This Tell Us?
Manual Connection: Both nodes are connected but have different roles: Node 1 is accepting an inbound connection from Node 2, and Node 2 is connecting manually to Node 1. The connection type "manual" confirms that you manually initiated the connection from Node 2 to Node 1 using addnode.
Syncing and Block Information: The synced_headers and synced_blocks values show the syncing progress. Node 2 is more in sync with the blockchain than Node 1 (309 blocks vs -1), which suggests that Node 1 might still be in the process of syncing or not yet fully synced.
Connection Health: The pingtime and other message statistics indicate that the connection is healthy and data is being exchanged.

That's why:
```bash
$ bitcoin-cli -datadir=$BITCOIN1_DIR getaddednodeinfo
[
  {
    "addednode": "127.0.0.1:18445",
    "connected": false,
    "addresses": [
    ]
  },
  {
    "addednode": "127.0.0.1:8334",
    "connected": false,
    "addresses": [
    ]
  }
]
```

```bash
$ bitcoin-cli -datadir=$BITCOIN2_DIR getaddednodeinfo
[
  {
    "addednode": "127.0.0.1:8333",
    "connected": true,
    "addresses": [
      {
        "address": "127.0.0.1:8333",
        "connected": "outbound"
      }
    ]
  }
]
```
