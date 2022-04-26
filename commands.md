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

`cardano-cli query tip --testnet-magic 1097911063`

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

```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    $TESTNET
```

