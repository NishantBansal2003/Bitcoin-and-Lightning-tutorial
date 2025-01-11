# Simulate Lightning transaction on the Bitcoin regtest network
To be able to follow along, you must have both bitcoind and CLN installed.

## Create and pay an invoice with CLN
### Start a Lightning Network on regtest with two connected nodes
We are in the lightning repository.
First load this file up.
```bash
$ source contrib/startup_regtest.sh
lightning-cli is /Users/nishantbansal/Desktop/Open-Source/SoB/lightning/cli/lightning-cli
lightningd is /Users/nishantbansal/Desktop/Open-Source/SoB/lightning/lightningd/lightningd
lightning-dir is /tmp
bitcoin-cli is bitcoin-cli
bitcoind is bitcoind
bitcoin-dir is /Users/nishantbansal/Library/Application Support/Bitcoin/
Useful commands:
  start_ln 3: start three nodes, l1, l2, l3
  connect 1 2: connect l1 and l2
  fund_nodes: connect all nodes with channels, in a row
  stop_ln: shutdown
  destroy_ln: remove ln directories
```

So we run the following command to start a network with only 2 nodes:
```bash
$ start_ln 2
Bitcoin Core starting
awaiting bitcoind...
Making "default" bitcoind wallet.
[3] 56601
[4] 56604
WARNING: eatmydata not found: install it for faster testing
Commands: 
        l1-cli, l1-log,
        l2-cli, l2-log,
        bt-cli, stop_ln, fund_nodes
clnrest is disabled. Try installing python developer dependencies
with 'poetry install'
```
This creates a wallet called `default`.

And it provides the following aliases: `l1-cli`, `l1-log`, `l2-cli`, `l2-log`, `bt-cli`, `stop_ln` and `fund_nodes`.

**Optional:**
We can look at those aliases by running this commands:
```bash
$ alias | grep -E 'l[12]|bt'
bt-cli='"$BCLI" -datadir="$BITCOIN_DIR" -regtest'
l1-cli='/Users/nishantbansal/Desktop/Open-Source/SoB/lightning/cli/lightning-cli --lightning-dir=/tmp/l1'
l1-log='less /tmp/l1/log'
l2-cli='/Users/nishantbansal/Desktop/Open-Source/SoB/lightning/cli/lightning-cli --lightning-dir=/tmp/l2'
l2-log='less /tmp/l2/log'
```

What we can do know is to get some information about regtest chain by running this command:
```bash
$ bt-cli -getinfo
Chain: regtest
Blocks: 776
Headers: 776
Verification progress: 100.0000%
Difficulty: 4.656542373906925e-10

Network: in 0, out 0, total 0
Version: 280000
Time offset (s): 0
Proxies: n/a
Min tx relay fee rate (BTC/kvB): 0.00001000

Wallet: default
Keypool size: 4000
Transaction fee rate (-paytxfee) (BTC/kvB): 0.00000000

Balance: 0.00000000

Warnings: (none)
```

Before connecting the nodes, we look at the node l1 by running the following command:
```bash
$ l1-cli getinfo
{
   "id": "03b8ff2dc194bbca66539898d56b3ddaefc3fe5535fe20608fc6762f8e082a9abd",
   "alias": "LIGHTNINGFEED-v24.11-30-g219623c",
   "color": "03b8ff",
   "num_peers": 0,
   "num_pending_channels": 0,
   "num_active_channels": 0,
   "num_inactive_channels": 0,
   "address": [],
   "binding": [
      {
         "type": "ipv4",
         "address": "127.0.0.1",
         "port": 7171
      }
   ],
   "version": "v24.11-30-g219623c",
   "blockheight": 776,
   "network": "regtest",
   "fees_collected_msat": 0,
   "lightning-dir": "/tmp/l1/regtest",
   "our_features": {
      "init": "8008a0882a8a59a1",
      "node": "8088a0882a8a59a1",
      "channel": "",
      "invoice": "02000002024100"
   }
}
```
We can do the same for the node l2 and we get the same kind of information.
```bash
$ l2-cli getinfo
{
   "id": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
   "alias": "UNITEDFARM-v24.11-30-g219623c",
   "color": "028097",
   "num_peers": 0,
   "num_pending_channels": 0,
   "num_active_channels": 0,
   "num_inactive_channels": 0,
   "address": [],
   "binding": [
      {
         "type": "ipv4",
         "address": "127.0.0.1",
         "port": 7272
      }
   ],
   "version": "v24.11-30-g219623c",
   "blockheight": 776,
   "network": "regtest",
   "fees_collected_msat": 0,
   "lightning-dir": "/tmp/l2/regtest",
   "our_features": {
      "init": "8008a0882a8a59a1",
      "node": "8088a0882a8a59a1",
      "channel": "",
      "invoice": "02000002024100"
   }
}
```

Using the command connect provided by the script `contrib/startup_regtest.sh` we connect the nodes l1 and l2.
```bash
$ connect 1 2
{
   "id": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
   "features": "8008a0882a8a59a1",
   "direction": "out",
   "address": {
      "type": "ipv4",
      "address": "127.0.0.1",
      "port": 7272
   }
}
```
Now, by running the command `l2-cli getinfo` and looking at the attribute num_peers we see that l2 is connected to another peer and that peer is the node l1.
We obtain a similar information for the node l1 by running the command:
```bash
$ l1-cli getinfo | jq .num_peers
1
```

### Fund a channel from the node l1 to the node l2
The two nodes connected, we can verify that there is no active channels between the nodes l1 and l2:
```bash
$ l1-cli listchannels
{
   "channels": []
}
$ l2-cli listchannels
{
   "channels": []
}
```
Before openning a channel between the nodes l1 and l2 we can check that the wallets of the node l1 and the node l2 has no coins:
```bash
$ l1-cli bkpr-listbalances
{
   "accounts": [
      {
         "account": "wallet",
         "balances": []
      }
   ]
}
$ l2-cli bkpr-listbalances
{
   "accounts": [
      {
         "account": "wallet",
         "balances": []
      }
   ]
}
```

Now we fund a channel from the node l1 to the node l2 using the command `fund_nodes` provided by the script `contrib/startup_regtest.sh`:
```bash
$ fund_nodes
bitcoind balance: 253.68754825
Waiting for lightning node funds... found.
Funding channel <-> node 1 to node 2. Waiting for confirmation... done.
```
If got some error mine some blocks using: `bt-cli -generate 200`

Using the listchannels sub command we can check that a channel has been funding from node l1 to l2
```bash
$ l1-cli listchannels | jq
{
  "channels": [
    {
      "source": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
      "destination": "03b8ff2dc194bbca66539898d56b3ddaefc3fe5535fe20608fc6762f8e082a9abd",
      "short_channel_id": "1079x1x0",
      "direction": 0,
      "public": true,
      "amount_msat": 1100000000,
      "message_flags": 1,
      "channel_flags": 0,
      "active": true,
      "last_update": 1736570283,
      "base_fee_millisatoshi": 1,
      "fee_per_millionth": 10,
      "delay": 6,
      "htlc_minimum_msat": 0,
      "htlc_maximum_msat": 1089000000,
      "features": ""
    },
    {
      "source": "03b8ff2dc194bbca66539898d56b3ddaefc3fe5535fe20608fc6762f8e082a9abd",
      "destination": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
      "short_channel_id": "1079x1x0",
      "direction": 1,
      "public": true,
      "amount_msat": 1100000000,
      "message_flags": 1,
      "channel_flags": 1,
      "active": true,
      "last_update": 1736570283,
      "base_fee_millisatoshi": 1,
      "fee_per_millionth": 10,
      "delay": 6,
      "htlc_minimum_msat": 0,
      "htlc_maximum_msat": 1089000000,
      "features": ""
    }
  ]
}
```
With the sub command bkpr-listbalances we can check that:
1. the wallet of the node l1 has some coins,

2. the node l1 has a channel opened with the node l2 with an amount of <some> msat on its side,

3. the node l2 has channel opened with the node l1 with an amount of <some> msat on its side:
```bash
$ l1-cli bkpr-listbalances
{
   "accounts": [
      {
         "account": "wallet",
         "balances": [
            {
               "balance_msat": 198999834000,
               "coin_type": "bcrt"
            }
         ]
      },
      {
         "account": "06faf727f5080d357aece97f6ac2b6ab5c62422662287ea8d844d4c103e5169f",
         "peer_id": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
         "we_opened": true,
         "account_closed": false,
         "account_resolved": false,
         "balances": [
            {
               "balance_msat": 1000000000,
               "coin_type": "bcrt"
            }
         ]
      }
   ]
}

$ l2-cli bkpr-listbalances
{
   "accounts": [
      {
         "account": "wallet",
         "balances": [
            {
               "balance_msat": 99899888000,
               "coin_type": "bcrt"
            }
         ]
      },
      {
         "account": "06faf727f5080d357aece97f6ac2b6ab5c62422662287ea8d844d4c103e5169f",
         "peer_id": "03b8ff2dc194bbca66539898d56b3ddaefc3fe5535fe20608fc6762f8e082a9abd",
         "we_opened": false,
         "account_closed": false,
         "account_resolved": false,
         "balances": [
            {
               "balance_msat": 100000000,
               "coin_type": "bcrt"
            }
         ]
      }
   ]
}
```

### Create an invoice with the node l2 and pay that invoice with the node l1
This is what we do by running the following command that generate a BOLT11 invoice:

- in the amount of 20000 sat,

- with the label inv-1 and

- the description pizza:

```bash
$ l2-cli invoice 20000sat inv-1 "pizza"                                                                                         ─╯
{
   "payment_hash": "33597e376923aa99e0753b20fc021e93628168a2ffdda65eed6adc52ba0218fc",
   "expires_at": 1737175666,
   "bolt11": "lnbcrt200u1pncraljsp5n5sjaprm24vj6f4uah6f60cnflsd9fuk9c9ucss3nd4sdqdjpyrspp5xdvhudmfyw4fncr48vs0cqs7jd3gz69zllw6vhhddtw99wszrr7qdqgwp5h57npxqyjw5qcqp2fp4pn74hrc6ng0x0y2mw7q7ldfcp08756wucmwgne0txnjkt4gummatq9qxpqysgqq540xejn2h4tx5ldh092frlju2fe80ndfzrxl336m7chq0k6mme4nyscmu9jraffv5nrvv6zfruyl4mszk47uzrrf4r2m27n9484xzcqh2mwue",
   "payment_secret": "9d212e847b55592d26bcedf49d3f134fe0d2a7962e0bcc42119b6b0681b20907",
   "created_index": 1,
   "warning_deadends": "Insufficient incoming capacity, once dead-end peers were excluded"
}
```

Using the value of the attribute bolt11 of the previous output, the node l1 will be able to pay that invoice. But before doing it, let's have a look at the invoices of the node l2 by running the following command:

```bash
$ l2-cli listinvoices
{
   "invoices": [
      {
         "label": "inv-1",
         "bolt11": "lnbcrt200u1pncraljsp5n5sjaprm24vj6f4uah6f60cnflsd9fuk9c9ucss3nd4sdqdjpyrspp5xdvhudmfyw4fncr48vs0cqs7jd3gz69zllw6vhhddtw99wszrr7qdqgwp5h57npxqyjw5qcqp2fp4pn74hrc6ng0x0y2mw7q7ldfcp08756wucmwgne0txnjkt4gummatq9qxpqysgqq540xejn2h4tx5ldh092frlju2fe80ndfzrxl336m7chq0k6mme4nyscmu9jraffv5nrvv6zfruyl4mszk47uzrrf4r2m27n9484xzcqh2mwue",
         "payment_hash": "33597e376923aa99e0753b20fc021e93628168a2ffdda65eed6adc52ba0218fc",
         "amount_msat": 20000000,
         "status": "unpaid",
         "description": "pizza",
         "expires_at": 1737175666,
         "created_index": 1
      }
   ]
}
```

By looking at the attribute `status` we can see that the invoice has not been paid yet.

Now let's pay that BOLT11 invoice by running the following command:
```bash
$ l1-cli pay lnbcrt200u1pncraljsp5n5sjaprm24vj6f4uah6f60cnflsd9fuk9c9ucss3nd4sdqdjpyrspp5xdvhudmfyw4fncr48vs0cqs7jd3gz69zllw6vhhddtw99wszrr7qdqgwp5h57npxqyjw5qcqp2fp4pn74hrc6ng0x0y2mw7q7ldfcp08756wucmwgne0txnjkt4gummatq9qxpqysgqq540xejn2h4tx5ldh092frlju2fe80ndfzrxl336m7chq0k6mme4nyscmu9jraffv5nrvv6zfruyl4mszk47uzrrf4r2m27n9484xzcqh2mwue
{
   "destination": "028097f881255dad407caafdd33f34865eb3b83266118d577507a96be4924c8b32",
   "payment_hash": "33597e376923aa99e0753b20fc021e93628168a2ffdda65eed6adc52ba0218fc",
   "created_at": 1736571012.752493000,
   "parts": 1,
   "amount_msat": 20000000,
   "amount_sent_msat": 20000000,
   "payment_preimage": "1092b5abf8196ab6cf3de1604c6072e978d15abe656375ccc0e65b89fad724d0",
   "status": "complete"
}
```

We can list the invoices of the node l2 using `l2-cli listinvoices` and verify looking at the attribute `paid_at` that the invoice labeled inv-1 has been paid at epoch unix time `1736571012`

We can check that the balances of the channel between the node l1 and l2 increased from 0 msat to 20000000 msat on the l2 side using:
`l2-cli bkpr-listbalances`
In the same vein, we can check that the balances of the channel between the node l1 and l2 decreased:
`l1-cli bkpr-listbalances`

### Closing the channels and the nodes
Now the node l1 initiate a mutual close of the channel `1079x1x0` it has oppened with the node l2 running the following command:
```bash
$ l1-cli close 1079x1x0
# Sending closing fee offer 195sat, with range 195sat-195sat
# Received closing fee offer 195sat, with range 195sat-1100000sat
{
   "txs": [
      "02000000000101542cc03e394a575378bb91f3f56bd566b51df98986c352b9646b48c9b51cf3390000000000ffffffff02c0d40100000000002251200fd01362997e7590566830af8fa11a65a0f6f2eefb59087de6d366cbbf8c90125df30e0000000000225120196cbbacae06debd6a36805dd0c2c58e9b7ee59d52d54dd829c0466d7035d21c040047304402200a92b2f1236f214f9739a28c5e41d3feb65f000990e3ba2f5c3884d549d2fc9302202df4daeacde3d6c4606e13089c85c9d0261958061064289f88a58c6fd1a702500147304402201c7bba8acc787eb69c94e130ba6b66bf8d583665ed0d452e686611c78ab5488302207f5435848eba1cc6fce93bd78dcdf71e2b3d55050b9357662c4a38033be29ad101475221023fa4b8f0e81c1749ab80a0e0606122a1a3be8e64ca3f9141dae8c82029d4338f21036909ad3bd10d6b0b0c748ab2333cba3d071a882534ccf83e1f0fb77f534c5e8b52ae00000000"
   ],
   "txids": [
      "11040c6dc3e54265505b314fcaa45ac386616da0247e1c4f004f821f3cf00766"
   ],
   "type": "mutual"
}
```

Since no block including that transaction has been mined so far, the channel still exist as we can see using:`l1-cli listchannels`

As we are doing a mutual close, we just have to mine one block on the regtest chain to close the channel: `bitcoin-cli -regtest -rpcwallet=default -generate 1`

And we can check that the channel is no longer listed on the network by running the following command:
```bash
$ l1-cli listchannels
{
   "channels": []
}
```


## Bonus(Doing ourselve instead of helper functions)
**Source:** https://lnroom.live/2022-11-08-lnroom-0001-create-and-pay-an-invoice-with-cln

- Things we need to start a Lightning node ourselves.
**Source:** https://lnroom.live/2022-11-22-lnroom-0002-start-a-lightning-node-on-regtest-with-cln/

- Connect Lightning nodes on regtest with CLN ourselves, and also introduce the jq command, a JSON processor.
**Source:** https://lnroom.live/2022-11-25-lnroom-0003-connect-lightning-nodes-on-regtest-with-cln/

- Fund the wallet of a CLN Lightning node running on regtest ourelves.
**Source:** https://lnroom.live/2022-11-29-lnroom-0004-fund-the-wallet-of-a-cln-lightning-node-running-on-regtest/

- Open a channel between two nodes running on regtest using CLN ourslves.
**Source:** https://lnroom.live/2022-12-02-lnroom-0005-open-a-channel-between-two-nodes-running-on-regtest-using-cln/

- Close payment channels of lightning nodes running on regtest with CLN ourselves.
**Source:** https://lnroom.live/2022-12-06-lnroom-0006-close-payment-channels-of-lightning-nodes-running-on-regtest-with-cln/

### Optional
- List payments of a CLN lightning node greater than 20000sat.
**Source:** https://lnroom.live/2022-12-09-lnroom-0007-list-payments-of-a-cln-lightning-node-greater-than-20000sat/

- A penalty transaction managed by Core Lightning.
**Source:** https://lnroom.live/2022-12-23-lnroom-0008-a-penalty-transaction-managed-by-core-lightning/

- Another penalty transaction on regtest with Core Lightning.
**Source:** https://lnroom.live/2022-12-27-lnroom-0009-another-penalty-transaction-on-regtest-with-core-lightning/
