```bash
bash -i <(curl -s https://install.aztec.network)
```
```
cd ~/.aztec
nano .env
```
```
ETHEREUM_HOSTS=http://your-ethereum-node:8545
L1_CONSENSUS_HOST_URLS=http://your-consensus-node:5052
VALIDATOR_PRIVATE_KEY=0x_your_actual_private_key_here
COINBASE=0x_your_actual_coinbase_address_here
P2P_IP=xxx.xxx.xxx.xxx
P2P_PORT=40401
PORT=8081
VALIDATOR_PRIVATE_KEY_1=0x_your_actual_key_1
VALIDATOR_PRIVATE_KEY_2=0x_your_actual_key_2
VALIDATOR_PRIVATE_KEY_3=0x_your_actual_key_3
VALIDATOR_PRIVATE_KEY_4=0x_your_actual_key_4
```
```bash
USER_NAME=$(whoami)
HOME_DIR=$(eval echo ~$USER_NAME)

sudo tee /etc/systemd/system/aztec.service > /dev/null <<EOF
[Unit]
Description=Aztec Validator Node
After=network.target docker.service
Requ
ires=docker.service

[Service]
Type=simple
User=$USER_NAME
WorkingDirectory=$HOME_DIR/.aztec
EnvironmentFile=$HOME_DIR/.aztec/.env
ExecStart=$HOME_DIR/.aztec/bin/aztec start --node --archiver --sequencer \\
  --network alpha-testnet \\
  --l1-rpc-urls \${ETHEREUM_HOSTS} \\
  --l1-consensus-host-urls \${L1_CONSENSUS_HOST_URLS} \\
  --sequencer.validatorPrivateKeys \${VALIDATOR_PRIVATE_KEY},\${VALIDATOR_PRIVATE_KEY_1},\${VALIDATOR_PRIVATE_KEY_2},\${VALIDATOR_PRIVATE_KEY_3},\${VALIDATOR_PRIVATE_KEY_4} \\
  --sequencer.publisherPrivateKey \${VALIDATOR_PRIVATE_KEY} \\
  --sequencer.coinbase \${COINBASE} \\
  --p2p.p2pIp \${P2P_IP} \\
  --p2p.p2pPort \${P2P_PORT} \\
  --port \${PORT}
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl start aztec.service
sudo systemctl enable aztec.service
sudo systemctl status aztec.service
```
Check logs:
```
journalctl -u aztec.service -f
```
```bash
chmod 600 ~/.aztec/.env
chown $USER_NAME:$USER_NAME ~/.aztec/.env
```
