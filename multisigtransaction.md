# Multi-signature



#### Hashing public key
```bash
cardano-cli address key-hash \
    --payment-verification-key-file payment.vkey \
    --out-file payment.hkey 
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

#### 2. Create the address
```bash
cardano-cli address build \
--payment-script-file multisignaturepolicy.script \
$TESTNET \
--out-file multisig.addr
```

```bash
export fee=0
export amount=100000000
export output=10000000
export txhash="74751b1321908c55effc40384be061c6cc8872e932fd55dc2edc450fea7c5612"
export txix=0
export address=$TARGADDR
export changeAddr=$(< multisig.addr)
```

### Raw transaction
```bash
cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $address+$output \
 --tx-out $changeAddr+$amount \
 --metadata-json-file metadata.json \
 --out-file multisigtx.raw
```

### Calculating fees
```bash
export fee=$(cardano-cli transaction calculate-min-fee --tx-body-file matx.raw --tx-in-count 1 --tx-out-count 2 --witness-count 3 $TESTNET --protocol-params-file protocol.json | cut -d " " -f1)
```

### calculating rest
```bash
export changeAmount=$(expr $amount - $output - $fee)
```

```bash
cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $address+$output \
 --tx-out $changeAddr+$changeAmount \
 --metadata-json-file metadata.json \
 --out-file multisigtx.raw
```

#### Witnessing
One file for each witness

```bash
cardano-cli transaction witness \
--signing-key-file payment.skey \
--tx-body-file multisigtx.unsigned \
--out-file payment.witness
```

```bash
cardano-cli transaction witness \
--signing-key-file verifier1.skey \
--tx-body-file multisigtx.unsigned\
--out-file verifier1.witness
```

```bash
cardano-cli transaction witness \
--signing-key-file verifier2.skey \
--tx-body-file multisigtx.unsigned\
--out-file verifier2.witness
```

#### Assembling the transaction
cardano-cli transaction assemble \
--tx-body-file multisigtx.unsigned \
--witness-file payment.witness \
--witness-file verifier1.witness \
--witness-file verifier2.witness \
--out-file multisigtx.signed

#### Submitting
cardano-cli transaction submit $TESTNET --tx-file multisigtx.signed

