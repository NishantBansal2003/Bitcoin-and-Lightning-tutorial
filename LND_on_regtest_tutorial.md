# Simulate Lightning transaction on the Bitcoin regtest network
To be able to follow along, you must have both bitcoind and lnd installed.

## 📜 Introduction
The Lightning Network is a second-layer protocol built on top of the Bitcoin blockchain. It enables faster and cheaper transactions by creating off-chain payment channels between users. These channels allow multiple transactions to occur without needing to be recorded on the main Bitcoin blockchain. Instead, only the opening and closing transactions of the channel are settled on the blockchain. This interaction reduces congestion on the Bitcoin network and enables microtransactions with almost instant settlement times.

### 📡 The Bitcoin testnet network mode
It’s a public test blockchain where the bitcoin does not have a real-world value that works similarly to the mainnet blockchain. In this way, it is safe to test out some functionality. 
Source: https://studygroup.moralis.io/

### 📡 The Bitcoin regtest network mode 👈🏿 (this is the mode we are going to be using)
Regtest (regression test) mode creates a local private blockchain where you can adjust the parameters to what you want. You usually use this when it is not needed to communicate with other peers and blocks. An example of what you can do with the parameters is you can create blocks instantly
Source: https://studygroup.moralis.io/

### 📡 The Bitcoin signet network mode
Signet is a proposed new test network parallel to the Bitcoin network. Like testnet and regtest, developers would use signet as a testing environment. Unlike the Bitcoin mainnet or the other test networks, signet would use digital signatures to validate blocks, not a Proof-of-Work system.
Source: https://river.com/

## ⚙️ Configuring our Bitcoin environment
In `~/Library/Application\ Support/Bitcoin/bitcoin.conf` add the below settings:
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

Start bitcoin service:
```bash
$ bitcoind
Bitcoin Core starting
```

ensure we are on regtest:
```bash
$ bitcoin-cli  getblockchaininfo
{
  "chain": "regtest",
  "blocks": 763,
  "headers": 763,
  "bestblockhash": "0f51afca9a4a72a36657774599e9c4809dfdace74a0370c2a5215ea9d7e5afd6",
  "difficulty": 4.656542373906925e-10,
  "time": 1735830872,
  "mediantime": 1735830871,
  "verificationprogress": 1,
  "initialblockdownload": false,
  "chainwork": "00000000000000000000000000000000000000000000000000000000000005f8",
  "size_on_disk": 230939,
  "pruned": false,
  "warnings": [
  ]
}
```

create/load a Bitcoin wallet:
```bash
$ bitcoin-cli createwallet "test-wallet-1"
```
OR
```bash
$ bitcoin-cli loadwallet "test-wallet-1"
{
  "name": "test-wallet-1"
}
```

lists all bitcoin wallets:
```bash
$ bitcoin-cli listwallets
[
  "test-wallet-1"
]
```

get a list of all our generated addresses:
```bash
$ bitcoin-cli listreceivedbyaddress 1 true
```

To generate some blocks use(Important):

This command mines 6 blocks but does not require specifying an address for the rewards. Instead, it sends the rewards to an address managed by the currently loaded wallet.

```bash
$ bitcoin-cli -generate 6
```
OR

This command generates 6 blocks and sends the block rewards (coinbase rewards) to the specified <generated-address>.
```bash
$ bitcoin-cli generatetoaddress 6 "<generated-address>"
```

Check our wallet info:
```bash
$ bitcoin-cli getwalletinfo
```

## ⚙️ Configuring our LND environment

### Configuring two LND nodes
```bash
$ mkdir /Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd1
$ mkdir /Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd2
$ export LND1_DIR="/Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd1"
$ export LND2_DIR="/Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd2"
$ export LNCLI1_MACAROONPATH="/Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd1/data/chain/bitcoin/regtest/admin.macaroon"
$ export LNCLI2_MACAROONPATH="/Users/nishantbansal/Desktop/Open-Source/SoB/LND-Tutorial/lnd2/data/chain/bitcoin/regtest/admin.macaroon"
```

In `$LND1_DIR/lnd.conf` add the below settings:
```bash
[Bitcoin]

bitcoin.active=1
bitcoin.regtest=1
bitcoin.node=bitcoind

[Bitcoind]

bitcoind.rpchost=127.0.0.1
bitcoind.rpcuser=nishant
bitcoind.rpcpass=nishant
bitcoind.zmqpubrawblock=tcp://127.0.0.1:28332
bitcoind.zmqpubrawtx=tcp://127.0.0.1:28333
```

In `$LND2_DIR/lnd.conf` add the below settings:
```bash
[Application Options]

listen=0.0.0.0:9734
rpclisten=127.0.0.1:11009
restlisten=0.0.0.0:8180

[Bitcoin]

bitcoin.active=1
bitcoin.regtest=1
bitcoin.node=bitcoind

[Bitcoind]

bitcoind.rpchost=127.0.0.1
bitcoind.rpcuser=nishant
bitcoind.rpcpass=nishant
bitcoind.zmqpubrawblock=tcp://127.0.0.1:28332
bitcoind.zmqpubrawtx=tcp://127.0.0.1:28333
```

**Quick note:** Notice that the config above has different ports and settings for network, RPC, and REST connections configured. well, to mention a few reasons:

Each LND instance needs to listen on different ports to avoid conflicts.

We are currently in `/Users/nishantbansal/Desktop/Open-Source/SoB/LND Bin/bin` directory

1. Start the bitcoind backend: `$ bitcoind`
2. Start up our first LND node: `$ ./lnd --lnddir=$LND1_DIR`
3. Now create/load a wallet: `$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH create` OR `$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH unlock`
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH getinfo                                                           ─╯
{
    "version":  "0.18.99-beta commit=kvdb/v1.4.12-176-g3d19e1976-dirty",
    "commit_hash":  "3d19e1976d27cb0e53100af0fc4ff0248924622b",
    "identity_pubkey":  "033d08bd50cc968ede5fdac0581bb0e02cfad22ccd272504b8633c283a9102730e",
    "alias":  "033d08bd50cc968ede5f",
    "color":  "#3399ff",
    "num_pending_channels":  0,
    "num_active_channels":  0,
    "num_inactive_channels":  0,
    "num_peers":  0,
    "block_height":  763,
    "block_hash":  "0f51afca9a4a72a36657774599e9c4809dfdace74a0370c2a5215ea9d7e5afd6",
    "best_header_timestamp":  "1735830872",
    "synced_to_chain":  false,
    "synced_to_graph":  false,
    "testnet":  false,
    "chains":  [
        {
            "chain":  "bitcoin",
            "network":  "regtest"
        }
    ],
    "uris":  [],
    "features":  {
        "0":  {
.......
```
4. Time to create an address that would receive the bitcoin sent to the lightning node wallet:
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH newaddress np2wkh
{
    "address":  "2NCdK2fuZ6FW8Le9A9fKkupprWZeEJo6AEi"
}
```
5. Let's load up our Bitcoin wallet, send some(100) Bitcoin from it to our lightning address above and then generate some blocks to give it some confirmations(first time only):
Run the commands below in sequence:
```bash
$ bitcoin-cli loadwallet "test-wallet-1"
{
  "name": "test-wallet-1"
}
```
Since our wallet is encrypted we need to execute below command:
```bash
$ bitcoin-cli walletpassphrase "test-wallet-1" 10000
```

```bash
$ bitcoin-cli sendtoaddress 2NCdK2fuZ6FW8Le9A9fKkupprWZeEJo6AEi 100
3496630a89fdadfb5cdd0e0f91ee378061f3f847038de9a4814713b26791f4cb
```

Now need to mine some blocks to confirm this transaction(otherwise BTC will be in unconfirmed_balance)
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH walletbalance
{
    "total_balance":  "10000000000",
    "confirmed_balance":  "0",
    "unconfirmed_balance":  "10000000000",
    "locked_balance":  "0",
    "reserved_balance_anchor_chan":  "0",
    "account_balance":  {
        "default":  {
            "confirmed_balance":  "0",
            "unconfirmed_balance":  "10000000000"
        }
    }
}
```
```bash
$ bitcoin-cli -generate 6
{
  "address": "bcrt1qtvt3yds8a50nyz9qpzz887vh53yhkh6acle8xy",
  "blocks": [
    "4f7691c30aab3586a4d15862f27283cf99145286ba149b00bea8680f023b4e28",
    "1c164fb2813fe158c173d608f7ab33415a8a6d00b9f6ec5fe97395602feef9e5",
    "23e36974a8ce5d8f5fd2dba0c44d663d7d52e99bed04d7c5e3abc7bc7146beed",
    "19ec1356bd8055e2b8b5e20cf47ffec794badc237d0c066e90c85dd03e00981c",
    "7f1cf2903e667a3f18e319d9f60e38d56807f63bd4fffa48b9a59fdcb9bf90f2",
    "51fe102362fca69c9292ea15a71f5fc644dd05883189dce76bdec58b76d2f728"
  ]
}
```

Let's go ahead and check our lnd1 wallet balance
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH walletbalance
{
    "total_balance":  "10000000000",
    "confirmed_balance":  "10000000000",
    "unconfirmed_balance":  "0",
    "locked_balance":  "0",
    "reserved_balance_anchor_chan":  "0",
    "account_balance":  {
        "default":  {
            "confirmed_balance":  "10000000000",
            "unconfirmed_balance":  "0"
        }
    }
}
```

6. Start up our second LND node: `$ ./lnd --lnddir=$LND2_DIR --adminmacaroonpath=$LND2_DIR/data/chain/bitcoin/regtest/admin.macaroon`
7. Now create/load a wallet: `$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 create` OR `$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 unlock`
```bash
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 getinfo                               ─╯
{
    "version":  "0.18.99-beta commit=kvdb/v1.4.12-176-g3d19e1976-dirty",
    "commit_hash":  "3d19e1976d27cb0e53100af0fc4ff0248924622b",
    "identity_pubkey":  "02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc",
    "alias":  "02915c080f3acfb2a0aa",
    "color":  "#3399ff",
    "num_pending_channels":  0,
    "num_active_channels":  0,
    "num_inactive_channels":  0,
    "num_peers":  0,
    "block_height":  769,
    "block_hash":  "51fe102362fca69c9292ea15a71f5fc644dd05883189dce76bdec58b76d2f728",
    "best_header_timestamp":  "1735890172",
    "synced_to_chain":  true,
    "synced_to_graph":  false,
    "testnet":  false,
    "chains":  [
        {
            "chain":  "bitcoin",
            "network":  "regtest"
        }
    ],
    "uris":  [],
    "features":  {
        "0":  {
...............
```

### 📜 Connect the lightning nodes to form a peer
Now that both of our nodes are ready for some action, we need to connect them to form a peer-to-peer network this would enable our nodes to communicate with each other on the network by opening and closing channels.

From lnd1 run:
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH listpeers
{
    "peers":  [] //👈🏿 no peers for now...let's fix that.
}
```

Now let's connect lnd2 to lnd1 by passing the `identity_pubkey`, `IP address` and `port`
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH connect 02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc@localhost:9734
{
    "status":  "connection to 02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc@127.0.0.1:9734 initiated"
}
```
Now Let's run and check
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH listpeers                                                         ─╯
{
    "peers":  [
        {
            "pub_key":  "02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc",
            "address":  "127.0.0.1:9734",
            "bytes_sent":  "394",
            "bytes_recv":  "394",
            "sat_sent":  "0",
            "sat_recv":  "0",
            "inbound":  false,
            "ping_time":  "-1",
            "sync_type":  "ACTIVE_SYNC",
            "features":  {
                "0":  {
                    "name":  "data-loss-protect",
                    "is_required":  true,
                    "is_known":  true
                },
                "5":  {
                    "name":  "upfront-shutdown-script",
                    "is_required":  false,
                    "is_known":  true
                },
                "7":  {
                    "name":  "gossip-queries",
                    "is_required":  false,
                    "is_known":  true
                },
                "8":  {
                    "name":  "tlv-onion",
                    "is_required":  true,
                    "is_known":  true
                },
                "12":  {
                    "name":  "static-remote-key",
                    "is_required":  true,
                    "is_known":  true
                },
                "14":  {
                    "name":  "payment-addr",
                    "is_required":  true,
                    "is_known":  true
                },
                "17":  {
                    "name":  "multi-path-payments",
                    "is_required":  false,
                    "is_known":  true
                },
                "23":  {
                    "name":  "anchors-zero-fee-htlc-tx",
                    "is_required":  false,
                    "is_known":  true
                },
                "25":  {
                    "name":  "route-blinding",
                    "is_required":  false,
                    "is_known":  true
                },
                "27":  {
                    "name":  "shutdown-any-segwit",
                    "is_required":  false,
                    "is_known":  true
                },
                "31":  {
                    "name":  "amp",
                    "is_required":  false,
                    "is_known":  true
                },
                "45":  {
                    "name":  "explicit-commitment-type",
                    "is_required":  false,
                    "is_known":  true
                },
                "2023":  {
                    "name":  "script-enforced-lease",
                    "is_required":  false,
                    "is_known":  true
                }
            },
            "errors":  [],
            "flap_count":  1,
            "last_flap_ns":  "1735890801559611000",
            "last_ping_payload":  ""
        }
    ]
}
```
🎉 Both Nodes are now successfully connected as a peer on our local lightning network.

***Quick note:*** A payment channel on the lightning network is a two-way connection between two parties (lightning nodes) that enables them to exchange bitcoin.

To create a payment channel between lnd1 and lnd2, we'll be making use of the `pub_key` of lnd2 and we'll then be funding the channel with 100,000 SAT. the 100,000 SAT is lnd1's contribution to the channel
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH openchannel 02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc 100000
{
    "funding_txid": "dcd2988ce99386df0832c9cf6f5d53e422c6d905b6cc6492bac909a3831d0399"
}
```
We need to mine some blocks to confirm this transaction else:
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH listchannels
{
    "channels":  []
}
```

Once the channel has been created and lnd1's BTC contribution has been added to the channel, we need to mine/generate some blocks to increase confirmations for the Channel opening transaction.
```bash
$ bitcoin-cli -generate 6
```

```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH listchannels                                                      ─╯
{
    "channels": [
        {
            "active": true,
            "remote_pubkey": "02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc",
            "channel_point": "dcd2988ce99386df0832c9cf6f5d53e422c6d905b6cc6492bac909a3831d0399:0",
            "chan_id": "846623953453056",
            "capacity": "100000",
            "local_balance": "96530",
            "remote_balance": "0",
            "commit_fee": "3140",
            "commit_weight": "772",
            "fee_per_kw": "2500",
            "unsettled_balance": "0",
            "total_satoshis_sent": "0",
            "total_satoshis_received": "0",
            "num_updates": "0",
            "pending_htlcs": [],
            "csv_delay": 144,
            "private": false,
            "initiator": true,
            "chan_status_flags": "ChanStatusDefault",
            "local_chan_reserve_sat": "1000",
            "remote_chan_reserve_sat": "1000",
            "static_remote_key": false,
            "commitment_type": "ANCHORS",
            "lifetime": "2",
            "uptime": "2",
            "close_address": "",
            "push_amount_sat": "0",
            "thaw_height": 0,
            "local_constraints": {
                "csv_delay": 144,
                "chan_reserve_sat": "1000",
                "dust_limit_sat": "354",
                "max_pending_amt_msat": "99000000",
                "min_htlc_msat": "1",
                "max_accepted_htlcs": 483
            },
            "remote_constraints": {
                "csv_delay": 144,
                "chan_reserve_sat": "1000",
                "dust_limit_sat": "354",
                "max_pending_amt_msat": "99000000",
                "min_htlc_msat": "1",
                "max_accepted_htlcs": 483
            },
            "alias_scids": [],
            "zero_conf": false,
            "zero_conf_confirmed_scid": "0",
            "peer_alias": "unable to lookup peer alias: alias for node not found",
            "peer_scid_alias": "0",
            "memo": "",
            "custom_channel_data": "",
            "channel_id": "99031d83a309c9ba9264ccb605d9c622e4535d6fcfc93208df8693e98c98d2dc",
            "short_chan_id": "846623953453056",
            "short_chan_id_str": "770x1x0"
        }
    ]
}
```

**NOTE:** Since both nodes are already connected we can open channel from either side.

Now that we have confirmed that our channel is ready for transactions, let's go ahead and create a payable invoice for an amount of 50,000 SAT on lnd2
```bash
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 addinvoice --amt 50000
{
    "r_hash":  "d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b",
    "payment_request":  "lnbcrt500u1pnh0xflpp56w08q26t896vg2e9mtdkrem320tpwws9z9sfr7dw86dx97d90u4sdqqcqzzsxqyz5vqsp5z9945kvfy5g9afmakzyrur2t4hhn2tr87un8j0r0e6l5m5zm0fus9qxpqysgqk98c6j7qefdpdmzt4g6aykds4ydvf2x9lpngqcfux3hv8qlraan9v3s9296r5w5eh959yzadgh5ckgjydgyfxdpumxtuk3p3caugmlqpz5necs",
    "add_index":  "1",
    "payment_addr":  "114b5a598925105ea77db0883e0d4badef352c67f726793c6fcebf4dd05b7a79"
}
```

Let's quickly inspect or confirm the details of this newly created invoice from lnd1 by making use of the invoice's `payment_request`
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH decodepayreq lnbcrt500u1pnh0xflpp56w08q26t896vg2e9mtdkrem320tpwws9z9sfr7dw86dx97d90u4sdqqcqzzsxqyz5vqsp5z9945kvfy5g9afmakzyrur2t4hhn2tr87un8j0r0e6l5m5zm0fus9qxpqysgqk98c6j7qefdpdmzt4g6aykds4ydvf2x9lpngqcfux3hv8qlraan9v3s9296r5w5eh959yzadgh5ckgjydgyfxdpumxtuk3p3caugmlqpz5necs
{
    "destination":  "02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc",
    "payment_hash":  "d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b",
    "num_satoshis":  "50000",
    "timestamp":  "1735891263",
    "expiry":  "86400",
    "description":  "",
    "description_hash":  "",
    "fallback_addr":  "",
    "cltv_expiry":  "80",
    "route_hints":  [],
    "payment_addr":  "114b5a598925105ea77db0883e0d4badef352c67f726793c6fcebf4dd05b7a79",
    "num_msat":  "50000000",
    "features":  {
        "8":  {
            "name":  "tlv-onion",
            "is_required":  true,
            "is_known":  true
        },
        "14":  {
            "name":  "payment-addr",
            "is_required":  true,
            "is_known":  true
        },
        "17":  {
            "name":  "multi-path-payments",
            "is_required":  false,
            "is_known":  true
        },
        "25":  {
            "name":  "route-blinding",
            "is_required":  false,
            "is_known":  true
        }
    },
    "blinded_paths":  []
}
```

📜 Pay or satisfy the invoice created by lnd2 via lnd1
Time for lnd1 to pay up 🧾
Run the command below and follow the prompts to confirm payment:
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH payinvoice lnbcrt500u1pnh0xflpp56w08q26t896vg2e9mtdkrem320tpwws9z9sfr7dw86dx97d90u4sdqqcqzzsxqyz5vqsp5z9945kvfy5g9afmakzyrur2t4hhn2tr87un8j0r0e6l5m5zm0fus9qxpqysgqk98c6j7qefdpdmzt4g6aykds4ydvf2x9lpngqcfux3hv8qlraan9v3s9296r5w5eh959yzadgh5ckgjydgyfxdpumxtuk3p3caugmlqpz5necs
Payment hash: d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b
Description: 
Amount (in satoshis): 50000
Fee limit (in satoshis): 2500
Destination: 02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc
Confirm payment (yes/no): yes
+------------+--------------+--------------+--------------+-----+----------+-----------------+----------------------+
| HTLC_STATE | ATTEMPT_TIME | RESOLVE_TIME | RECEIVER_AMT | FEE | TIMELOCK | CHAN_OUT        | ROUTE                |
+------------+--------------+--------------+--------------+-----+----------+-----------------+----------------------+
| SUCCEEDED  |        0.059 |        0.332 | 50000        | 0   |      858 | 846623953453056 | 02915c080f3acfb2a0aa |
+------------+--------------+--------------+--------------+-----+----------+-----------------+----------------------+
Amount + fee:   50000 + 0 sat
Payment hash:   d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b
Payment status: SUCCEEDED, preimage: bd3e3c19bf04b4a1478abf7b9e0f9896d6d817c2312923f7df441365862bed36
```

Now that we have made payment, let's look up the invoice on lnd2 we'll be using the `payment hash` above
```bash
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 lookupinvoice d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b
{
    "memo":  "",
    "r_preimage":  "bd3e3c19bf04b4a1478abf7b9e0f9896d6d817c2312923f7df441365862bed36",
    "r_hash":  "d39e702b4b3974c42b25dadb61e77153d6173a05116091f9ae3e9a62f9a57f2b",
    "value":  "50000",
    "value_msat":  "50000000",
    "settled":  true,// 👈🏿 the invoice has been fulfilled ✅
    "creation_date":  "1735891263",
    "settle_date":  "1735891528",
    "payment_request":  "lnbcrt500u1pnh0xflpp56w08q26t896vg2e9mtdkrem320tpwws9z9sfr7dw86dx97d90u4sdqqcqzzsxqyz5vqsp5z9945kvfy5g9afmakzyrur2t4hhn2tr87un8j0r0e6l5m5zm0fus9qxpqysgqk98c6j7qefdpdmzt4g6aykds4ydvf2x9lpngqcfux3hv8qlraan9v3s9296r5w5eh959yzadgh5ckgjydgyfxdpumxtuk3p3caugmlqpz5necs",
    "description_hash":  "",
    "expiry":  "86400",
    "fallback_addr":  "",
    "cltv_expiry":  "80",
    "route_hints":  [],
    "private":  false,
    "add_index":  "1",
    "settle_index":  "1",
    "amt_paid":  "50000000",
    "amt_paid_sat":  "50000",
    "amt_paid_msat":  "50000000",
    "state":  "SETTLED",
    "htlcs":  [
        {
            "chan_id":  "846623953453056",
            "htlc_index":  "0",
            "amt_msat":  "50000000",
            "accept_height":  775,
            "accept_time":  "1735891528",
            "resolve_time":  "1735891528",
            "expiry_height":  858,
            "state":  "SETTLED",
            "custom_records":  {
                "106823":  "00"
            },
            "mpp_total_amt_msat":  "50000000",
            "amp":  null,
            "custom_channel_data":  "fe0001a1470100"
        }
    ],
    "features":  {
        "8":  {
            "name":  "tlv-onion",
            "is_required":  true,
            "is_known":  true
        },
        "14":  {
            "name":  "payment-addr",
            "is_required":  true,
            "is_known":  true
        },
        "17":  {
            "name":  "multi-path-payments",
            "is_required":  false,
            "is_known":  true
        },
        "25":  {
            "name":  "route-blinding",
            "is_required":  false,
            "is_known":  true
        }
    },
    "is_keysend":  false,
    "payment_addr":  "114b5a598925105ea77db0883e0d4badef352c67f726793c6fcebf4dd05b7a79",
    "is_amp":  false,
    "amp_invoice_state":  {},
    "is_blinded":  false,
    "blinded_path_config":  null
}
```
We are finally done with all lightning transactions so let's go ahead and close the payment channel and kill all running lightning and Bitcoin daemons as we close for the day.

Run the command below from any of the two nodes
To close all existing channels:
```bash
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 closeallchannels 
```
Before closing a channel, you need to identify its details:

Command:
```bash
$ lncli listchannels
```
Output: This will list all active channels, including their `channel_point`, which uniquely identifies the channel. The channel_point consists of the funding transaction ID and the output index, e.g., `txid:0`.
```bash
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 closechannel --funding_txid=dcd2988ce99386df0832c9cf6f5d53e422c6d905b6cc6492bac909a3831d0399 --output_index=0
{
    "closing_txid": "0908de2c35b637e16d181bbd50ebcf2265b642cb7c43e6126a6af1d8e6133689"
}
```

After initiating the closure, monitor the transaction on-chain to ensure it gets confirmed:
This shows the status of closing channels.

Once the closure transaction is confirmed, the channel funds will appear in your wallet.
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH pendingchannels                                                                  ─╯
{
    "total_limbo_balance":  "46530",
    "pending_open_channels":  [],
    "pending_closing_channels":  [],
    "pending_force_closing_channels":  [],
    "waiting_close_channels":  [
        {
            "channel":  {
                "remote_node_pub":  "02915c080f3acfb2a0aa51f758507dd8ed77337b38826a2ff10ad85d41336a02fc",
                "channel_point":  "dcd2988ce99386df0832c9cf6f5d53e422c6d905b6cc6492bac909a3831d0399:0",
                "capacity":  "100000",
                "local_balance":  "46530",
                "remote_balance":  "50000",
                "local_chan_reserve_sat":  "1000",
                "remote_chan_reserve_sat":  "1000",
                "initiator":  "INITIATOR_LOCAL",
                "commitment_type":  "ANCHORS",
                "num_forwarding_packages":  "2",
                "chan_status_flags":  "ChanStatusCoopBroadcasted|ChanStatusRemoteCloseInitiator",
                "private":  false,
                "memo":  "",
                "custom_channel_data":  ""
            },
            "limbo_balance":  "46530",
            "commitments":  {
                "local_txid":  "ee190d37268e91e9eeb1f493b3355c2dc8d8d2da0ac2409fea6ca1715883e2a5",
                "remote_txid":  "77dd3d20a850be06ac3b43d4fa396f59dcc5c1b953f2d8e6c48dd6a491b7b242",
                "remote_pending_txid":  "",
                "local_commit_fee_sat":  "2810",
                "remote_commit_fee_sat":  "2810",
                "remote_pending_commit_fee_sat":  "0"
            },
            "closing_txid":  "0908de2c35b637e16d181bbd50ebcf2265b642cb7c43e6126a6af1d8e6133689",
            "closing_tx_hex":  ""
        }
    ]
}
```

Now check the list of channels
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH listchannels                                                                     ─╯
{
    "channels":  []
}
```

Run the command below to kill all running daemons
```bash
$ ./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH stop
$ ./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009 stop
$ bitcoin-cli stop
```

## Extras
### aliases
```bash
#lnd1
alias lnd1="./lnd --lnddir=$LND1_DIR";
alias lncli1="./lncli --lnddir=$LND1_DIR --macaroonpath=$LNCLI1_MACAROONPATH"

#lnd2
alias lnd2="./lnd --lnddir=$LND2_DIR";
alias lncli2="./lncli --lnddir=$LND2_DIR --macaroonpath=$LNCLI2_MACAROONPATH --rpcserver=localhost:11009"
```

Taken from: 
- https://dev.to/shytypes1028/simulate-your-first-lightning-transaction-on-the-bitcoin-regtest-network-part-1-macos-4o3c
- https://dev.to/shytypes1028/simulate-your-first-lightning-transaction-on-the-bitcoin-regtest-network-part-2-macos-2el0
- https://dev.to/shytypes1028/simulate-your-first-lightning-transaction-on-the-bitcoin-regtest-network-part-3-macos-4b22

> **Author:** Nishant Bansal  
> **Date:** 2025-01-03
