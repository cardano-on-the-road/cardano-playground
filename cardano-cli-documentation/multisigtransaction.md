# Multi-signature transaction



#### Hashing public key
```bash
cardano-cli address key-hash \
    --payment-verification-key-file payment1.vkey \
    --out-file payment1.hkey 
```

#### 1) Multi-signature script file
The keyHash are generated from the public keys.
filename best practice: `multisignaturepolicy.script`
```json
{
    "type":"all",
    "scripts":[
        {
            "type":"sig",
            "keyHash":""
        },
        {
            "type":"sig",
            "keyHash":""
        },
        {
            "type":"sig",
            "keyHash":""
        }
    ]
}
```

cardano-cli query utxo \
    --address $(cat multisig.addr) \
    --testnet-magic=$MAGIC

#### 2. Create the address
```bash
cardano-cli address build \
--payment-script-file multisig.script \
--testnet-magic=$MAGIC \
--out-file multisig.addr
```

```
cardano-cli address build \
--payment-script-file ${POLICY_PATH}/policy.script \
--stake-verification-key-file ${KEYS_PATH}/stake.vkey \
${CARDANO_NET_PREFIX} \
--out-file ${ADDRESSES_PATH}/script-with-stake.addr
```

```bash
export fee=0
export amount=100000000
export output=10000000
export txhash="4a5d4091600a8ea3eac1c3cde32d99c4514541888b071007a69ccdcf6dfab639"
export txix=0
export address=$TARGADDR
export changeAddr=$(< multisig.addr)
```

### Transaction build
cardano-cli transaction build \
--tx-in $txhash#$txix  \
--tx-out $DSTADDR+$output \
--change-address $SRCADDR \
--tx-in-script-file multisig.script \
--witness-override 3 \
--metadata-json-file metadata.json \
--out-file multisigtx.unsigned \
--testnet-magic=$MAGIC



#### Witnessing
One file for each witness


```bash
cardano-cli transaction witness \
--signing-key-file payment1.skey \
--tx-body-file multisigtx.unsigned \
--out-file payment1.witness
```

```bash
cardano-cli transaction witness \
--signing-key-file payment2.skey \
--tx-body-file multisigtx.unsigned \
--out-file payment2.witness
```

```bash
cardano-cli transaction witness \
--signing-key-file payment3.skey \
--tx-body-file multisigtx.unsigned \
--out-file payment3.witnesss
```

#### Assembling the transaction
cardano-cli transaction assemble \
--tx-body-file multisigtx.unsigned \
--witness-file payment1.witness \
--witness-file payment2.witness \
--witness-file payment3.witness \
--out-file multisigtx.signed

#### Submitting
cardano-cli transaction submit --testnet-magic=$MAGIC --tx-file multisigtx.signed


# Transaction from 2 different address



