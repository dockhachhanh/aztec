# AZTEC SEQUENCER 1.2.1 SETUP GUIDE
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

---
# aztec
Aztec is a privacy-first Layer 2 on Ethereum. It supports smart contracts with both private &amp; public state and private &amp; public execution.
High level view

## Tổng quan hệ thống Aztec
- Người dùng tương tác với Aztec qua Aztec.js (giống như Web3.js hoặc Ethers.js).
- Private functions (hàm riêng tư) được thực thi trên thiết bị người dùng thông qua PXE.
- Public functions (hàm công khai) được thực thi bởi Public VM chạy trên các node mạng Aztec.
- Các giao dịch được rollup lại và gửi lên Ethereum để xác minh tính hợp lệ.

## 🔐 Private vs Public Execution
- Private execution: Thực hiện trên thiết bị người dùng thông qua PXE – môi trường thực thi riêng tư.
- Public execution: Thực hiện bởi các node mạng qua Public VM (Aztec VM).
- Luồng xử lý giao dịch: Private → Public (một chiều).
→ Private có thể gọi Public, nhưng Public không thể gọi ngược lại Private.

##🧠 PXE (Private Execution Environment)
- Chạy phía client (trình duyệt hoặc Node.js).
- Quản lý khóa, notes, tạo proof.
- Là phần của thư viện aztec.js.
- Không biết gì về Public VM.

### ⚙️ Public VM (Aztec VM)
- Tương tự EVM.
- Chạy trên các node.
- Không biết gì về PXE.

## 🌲 Trạng thái riêng tư và công khai
### Private state:
- Dùng mô hình UTXO với các "notes".
- Notes được lưu trong cây UTXO, và khi bị hủy sẽ tạo nullifier.
- Nullifiers lưu trong nullifier tree.
- Public state:
- Giống như Ethereum – sổ cái công khai.
- Dữ liệu lưu trong public data tree.

## 👛 Tài khoản và khóa
### Account abstraction:
- Mỗi tài khoản là một smart contract.
- Cho phép tùy biến xác thực, nonce, phí.
- Ba cặp khóa cho mỗi tài khoản:
- Nullifier key pair – dùng để hủy note.
- Incoming viewing key – mã hóa note cho người nhận.
- Outgoing viewing key – mã hóa note cho người gửi.
- Không có sẵn khóa ký – lập trình viên phải định nghĩa trong account contract.

## ✍️ Ngôn ngữ Noir
- DSL (domain-specific language) cho viết smart contract ZK.
- Có thể dùng để viết circuits xác minh onchain hoặc offchain.
- Noir được dùng để phát triển contract cho Aztec.
