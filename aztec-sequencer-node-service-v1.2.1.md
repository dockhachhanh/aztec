```
cd ~/.aztec
nano .env
```
```
ETHEREUM_HOSTS=
L1_CONSENSUS_HOST_URLS=
VALIDATOR_PRIVATE_KEY=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
COINBASE=0xCC9b931D13xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
P2P_IP=xxx.xxx.xxx.xxx
P2P_PORT=40401
PORT=8081
VALIDATOR_PRIVATE_KEY_1=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VALIDATOR_PRIVATE_KEY_2=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VALIDATOR_PRIVATE_KEY_3=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VALIDATOR_PRIVATE_KEY_4=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
```bash
USER_NAME=$(whoami)
HOME_DIR=$(eval echo ~$USER_NAME)

sudo tee /etc/systemd/system/aztec.service > /dev/null <<EOF
[Unit]
Description=Aztec Validator Node
After=network.target docker.service
Requires=docker.service

[Service]
Type=simple
User=$USER_NAME
WorkingDirectory=$HOME_DIR/.aztec
EnvironmentFile=$HOME_DIR/.aztec/.env
ExecStart=$HOME_DIR/.aztec/bin/aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls ${ETHEREUM_HOSTS} \
  --l1-consensus-host-urls ${L1_CONSENSUS_HOST_URLS} \
  --sequencer.validatorPrivateKeys ${VALIDATOR_PRIVATE_KEY},${VALIDATOR_PRIVATE_KEY_1},${VALIDATOR_PRIVATE_KEY_2},${VALIDATOR_PRIVATE_KEY_3},${VALIDATOR_PRIVATE_KEY_4} \
  --sequencer.publisherPrivateKey ${VALIDATOR_PRIVATE_KEY} \
  --sequencer.coinbase ${COINBASE} \
  --p2p.p2pIp ${P2P_IP} \
  --p2p.p2pPort ${P2P_PORT} \
  --port ${PORT}
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
