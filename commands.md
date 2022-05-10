## Git

```
git config --global user.name 'your username'
git config --global user.password 'your password'
git config --global user.email 'your email'


git config --local user.name 'your username'
git config --local user.password 'your password'
```

## Cardano usefull command

`sudo systemctl stop cardano-node`

`journalctl --unit=cardano-node --follow`

`cardano-cli query tip $TESTNET`

```bash
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

```bash
cardano-cli stake-address key-gen \
    --verification-key-file stake.vkey \
    --signing-key-file stake.skey
```

```bash
cardano-cli address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    --out-file payment.addr \
    $TESTNET
```

#### Hashing public key
```bash
cardano-cli address key-hash \
    --payment-verification-key-file payment.vkey \
    --out-file payment.hkey 
```

```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    $TESTNET
```

## Transactions

cardano-cli query protocol-parameters $TESTNET --out-file protocol.json

### Setting variables
```bash
export fee=0
export amount=200000000
export output=100000000
export txhash="369b2e6972d7b7de726972f8a2d2dce3539ae05bc4ac8d1adce1d3f86aff1f0d"
export txix=0
export address="addr_test1qp5h55qm05zhv6eapr3em26rurchp9eaxwvt9p69kwuvu42rj9ynfzwggc0s55nlpqegv90w2rshnqf0q3um5pytk4qqutq8j6"
```

### Raw transaction
```bash
cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $address+$output \
 --tx-out $MYADDR+$amount \
 --out-file matx.raw
```

### Transaction build (no Raw)
This one that calculates the fee, so the output is alreade an `.unsigned` file rather than an `.raw`

```bash
cardano-cli transaction build \
--alonzo-era \
--tx-in $txhash#$txix  \
--tx-out $address+$output \
--change-address $MYADDR \
--out-file matx.unsigned \
$TESTNET
```

### Calculating fees
```bash
export fee=$(cardano-cli transaction calculate-min-fee --tx-body-file matx.raw --tx-in-count 1 --tx-out-count 1 --witness-count 1 $TESTNET --protocol-params-file protocol.json | cut -d " " -f1)
```

### calculating rest
```bash
export rest=$(expr $amount - $output - $fee)
```

### Transaction to sign
```bash
cardano-cli transaction build-raw \
--fee $fee  \
--tx-in $txhash#$txix  \
--tx-out $address+$output \
--tx-out $MYADDR+$rest \
--out-file matx.raw
```

### Signing transaction
```bash
cardano-cli transaction sign  \
--signing-key-file payment.skey  \
$TESTNET --tx-body-file matx.raw  \
--out-file matx.signed
```

### Submitting transaction
```bash
cardano-cli transaction submit --tx-file matx.signed $TESTNET
```

