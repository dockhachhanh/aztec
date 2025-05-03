# HƯỚNG DẪN CÀI ĐẶT BEACON RPC VÀ SEPOLIA RPC
---
## Bước 1: Cài đặt Rust
Lighthouse được viết bằng Rust, nên cần cài Rust để biên dịch.
Cài đặt Rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
- Chọn tùy chọn 1 (mặc định) khi được hỏi.
- Cập nhật biến môi trường:
```bash
source $HOME/.cargo/env
```
- Cập nhật Rust:
```bash
rustup update
```
- Kiểm tra Rust:
```bash
rustc --version
cargo --version
```
- Nên thấy phiên bản Rust (ví dụ: rustc 1.81.0) và Cargo.
## Bước 2: Biên dịch Lighthouse
- Tải mã nguồn Lighthouse:
```bash
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
```
- Chuyển sang phiên bản ổn định (khuyến nghị):
     - Để tránh lỗi với phiên bản beta (v7.0.1), dùng nhánh ổn định:
```bash
git checkout stable
```
- Biên dịch Lighthouse với backtrace:
     - Chạy với RUST_BACKTRACE=1 để thấy chi tiết nếu lỗi:
```bash
RUST_BACKTRACE=1 make
```
Nếu biên dịch thành công, binary lighthouse sẽ nằm trong ~/.cargo/bin.
- Kiểm tra Lighthouse:
```bash
lighthouse --version
```
Nên thấy phiên bản (ví dụ: Lighthouse/v7.0.0).

Nếu lỗi vẫn xảy ra:
- Kiểm tra log lỗi chi tiết (với RUST_BACKTRACE=1).
- Đảm bảo tất cả gói ở bước 1 đã cài đặt.
- Thử xóa thư mục build và thử lại:
```bash
rm -rf target
make
```
## Bước 3: Cài đặt Geth
- Cài đặt Geth:
```bash
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt update
sudo apt install -y geth
```
- Kiểm tra phiên bản Geth:
```bash
geth version
```
Đảm bảo phiên bản là v1.10.x hoặc mới hơn (ví dụ: v1.15.10).

## Bước 4: Tạo tệp JWT
JWT là khóa xác thực để Geth và Lighthouse giao tiếp an toàn.
- Tạo thư mục:
```bash
mkdir -p $HOME/.lighthouse
```
- Tạo tệp JWT:
```bash
openssl rand -hex 32 > $HOME/.lighthouse/jwt.hex
```
Đặt quyền bảo mật:
```bash
chmod 600 $HOME/.lighthouse/jwt.hex
```
Kiểm tra tệp:
```bash
cat $HOME/.lighthouse/jwt.hex
```
Kết quả phải là:
```
d176xxxx....
```
Bước 5: Chạy Geth
Mở session screen:
```bash
screen -S geth
```
Chạy Geth:
```bash
geth --sepolia \
  --http \
  --http.addr 0.0.0.0 \
  --http.api eth,net,web3 \
  --authrpc.port 8551 \
  --authrpc.jwtsecret $HOME/.lighthouse/jwt.hex \
  --syncmode snap
```
Thoát screen:
``` Nhấn Ctrl+A, rồi D. ```

Kiểm tra Geth:
Kiểm tra cổng:
```bash
netstat -tuln | grep 8545
```
Nên thấy ``` 0.0.0.0:8545. ```
```bash
netstat -tuln | grep 8551
```
Nên thấy ``` 127.0.0.1:8551. ```

Kiểm tra RPC:
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545
```
Kiểm tra đồng bộ:
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
```
- Chờ đến khi trả về "false".

## Bước 6: Chạy Lighthouse
Mở session screen:
```bash
screen -S lighthouse
```
Chạy Lighthouse:
```bash
lighthouse beacon_node \
  --network sepolia \
  --execution-endpoint http://localhost:8551 \
  --execution-jwt-secret-key d176xxx...
  --http \
  --http-address 0.0.0.0 \
  --http-port 5052 \
  --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io
```
- sửa dòng này với key lấy ở bước trên ```bash cat $HOME/.lighthouse/jwt.hex ```--execution-jwt-secret-key d176xxx...
or
```bash
lighthouse beacon_node \
  --network sepolia \
  --execution-endpoint http://localhost:8551 \
  --execution-jwt /home/og/.lighthouse/jwt.hex \
  --http \
  --http-address 0.0.0.0 \
  --http-port 5052 \
  --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io
```

Thoát screen:
Nhấn ``` Ctrl+A, rồi D. ```

Kiểm tra Lighthouse:
Kiểm tra cổng:
```bash
netstat -tuln | grep 5052
```
- Nên thấy ``` 0.0.0.0:5052. ```

Kiểm tra Beacon RPC:
```bash
curl -X GET http://localhost:5052/eth/v1/node/version
```
## Bước 7: Thiết lập Firewall
Cài đặt UFW:
```bash
sudo apt install -y ufw
```
Mở cổng:
```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 8545
sudo ufw allow 8551
sudo ufw allow 5052
sudo ufw allow 40400
sudo ufw allow 8080
sudo ufw enable
```
Kiểm tra:
```bash
sudo ufw status
```
