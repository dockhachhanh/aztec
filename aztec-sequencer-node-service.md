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
---
Get Role
Go to the discord channel :[operators| start-here](https://discord.com/channels/1144692727120937080/1367196595866828982/1367323893324582954) and follow the prompts, You can continue the guide with my commands if you need help.

![image](https://github.com/user-attachments/assets/90e9d34e-724b-481a-b41f-69b1eb4c9f65)

Run this command:
```bash
#!/bin/bash

# Gọi API để lấy BLOCK_NUMBER
BLOCK_NUMBER=$(curl -s -X POST -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
  http://localhost:8081 | jq -r ".result.proven.number")

# Gọi API để lấy PROOF dùng BLOCK_NUMBER
PROOF=$(curl -s -X POST -H "Content-Type: application/json" \
  -d "{\"jsonrpc\":\"2.0\",\"method\":\"node_getArchiveSiblingPath\",\"params\":[\"$BLOCK_NUMBER\",\"$BLOCK_NUMBER\"],\"id\":1}" \
  http://localhost:8081 | jq -r ".result")

# In kết quả
echo "BLOCK_NUMBER: $BLOCK_NUMBER"
echo "PROOF: $PROOF"
```
Register with Discord**
* Type the following command in this Discord server: `/operator start`
* After typing the command, Discord will display option fields that look like this:
* `address`:            Your validator address (Ethereum Address)
* `block-number`:      Block number for verification (Block number from Step 1)
* `proof`:             Your sync proof (base64 string from Step 2)

Then you'll get your `Apprentice` Role
