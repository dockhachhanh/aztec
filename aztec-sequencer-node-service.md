Installation:
To use Aztec's suite of tools you'll need to:
Get docker (engine or desktop)
```bash
Run bash -i <(curl -s https://install.aztec.network)
```
Now install the latest testnet version of aztec: 
```
export PATH="$HOME/.aztec/bin:$PATH"
source ~/.bashrc
~/.aztec/bin/aztec-up -v 0.87.8
```
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
```
create aztec.service
```bash
sudo nano /etc/systemd/system/aztec.service
```
- Dán nội dung sau:
```bash
[Unit]
Description=Aztec Validator Node
After=network.target docker.service
Requires=docker.service

[Service]
Type=simple
User=$USER
WorkingDirectory=/home/$USER/.aztec
EnvironmentFile=/home/$USER/.aztec/.env
ExecStart=/home/$USER/.aztec/bin/aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls ${ETHEREUM_HOSTS} \
  --l1-consensus-host-urls ${L1_CONSENSUS_HOST_URLS} \
  --sequencer.validatorPrivateKey ${VALIDATOR_PRIVATE_KEY} \
  --sequencer.coinbase ${COINBASE} \
  --p2p.p2pIp ${P2P_IP} \
  --p2p.p2pPort 40401 \
  --port 8081
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target

```
start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable aztec
sudo systemctl start aztec
```
check status
```bash
sudo systemctl status aztec
```
check logs
```bash
sudo journalctl -fu aztec
```
