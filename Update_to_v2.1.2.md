```
sudo systemctl stop aztec

```
```
cd ~/.aztec/bin
./aztec-up
```
```
cd ~/.aztec/bin
./aztec -V
# will see 2.1.2
```
2. Install cast (Foundry tool)
If cast is not installed yet:
```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup
```
```bash
cast --version
```
. Check validator wallet balance
Make sure your validator wallet (the one holding ETH) has enough funds for gas.
```bash
source ~/.aztec/.env
cast balance $COINBASE --rpc-url $ETHEREUM_HOSTS
```
If the balance is low, send more ETH to that address.
4. Approve the Rollup Contract

Before adding a validator, you must approve the rollup contract to spend your stake tokens.
```bash
cast send 0x139d2a7a0881e16332d7D1F8DB383A4507E1Ea7A \
  "approve(address,uint256)" \
  ${ROLLUP} \
  200000ether \
  --private-key ${VALIDATOR_PRIVATE_KEY} \
  --rpc-url ${ETHEREUM_HOSTS}
```
This allows the Rollup contract to handle up to 200,000 tokens from your validator wallet.
5. Generate BLS key

Before adding your validator, you must create a BLS keypair for attestation.
This is different from your Ethereum validator key.

Option 1 — Generate new ETH + BLS keystore (recommended)
```bash
aztec validator-keys new \
  --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 \
  --data-dir ~/.aztec/keystore \
  --file key1.json
```
- This will create both Ethereum and BLS keys under ~/.aztec/keystore/key1.json.
- The command will also display a mnemonic phrase — save it securely.
```bash
aztec validator-keys new \
  --mnemonic "your twelve word mnemonic here" \
  --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 \
  --data-dir ~/.aztec/keystore \
  --file key1.json
```
If you already have a mnemonic and want to reuse it:

you will see result:
```
Wrote sequencer keystore to /Users/your-name/.aztec/keystore/key1.json

acc1:
  attester:
    eth: 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
    bls: 0x1a2b3c4d5e6f7890abcdef1234567890abcdef1234567890abcdef1234567890
```
6. Add validator to the network
Run the following command to register your validator:
If you are not a validator
```bash
cd ~/.aztec
source .env
./bin/aztec add-l1-validator \
  --l1-rpc-urls $ETHEREUM_HOSTS \
  --network testnet \
  --private-key $PRIVATE_KEY_OF_OLD_SEQUENCER \
  --attester $ETH_ATTESTER_ADDRESS \
  --withdrawer $ANY_ETH_ADDRESS \
  --bls-secret-key $BLS_ATTESTER_ADDRESS \
  --rollup $ROLLUP
```
