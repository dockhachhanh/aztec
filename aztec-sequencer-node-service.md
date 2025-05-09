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
ExecStart=$HOME/.aztec/bin/aztec start --node --archiver --sequencer --network alpha-testnet --p2p.maxPeers 100 --log-level debug
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/aztec-sequencer.service > /dev/null 
```
start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable aztec-sequencer
sudo systemctl start aztec-sequencer
```
check status
```bash
sudo systemctl status aztec-sequencer
```
check logs
```bash
sudo journalctl -fu aztec-sequencer
```
