Create .env
```bash
nano $HOME/.aztec/.env
```
Enter content
```
ETHEREUM_HOSTS=<http...> //sepolia rpc
L1_CONSENSUS_HOST_URLS=<http...> // beacon rpc
VALIDATOR_PRIVATE_KEY=0x<private key>
COINBASE=0x<wallet address>
P2P_IP= <public ip>
P2P_PORT=40400
LOG_LEVEL=debug
P2P_MAX_TX_POOL_SIZE=1000000000
P2P_MAX_PEERS=50
```
create aztec-sequencer.service
```bash
echo "[Unit]
Description=Aztec Sequencer Node
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
Group=$USER
WorkingDirectory=$HOME/.aztec
EnvironmentFile=$HOME/.aztec/.env
ExecStart=$HOME/.aztec/bin/aztec start --node --archiver --sequencer --network alpha-testnet --p2p.maxPeers --log-level
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/aztec-sequencer.service > /dev/null 
```
