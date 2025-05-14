# HƯỚNG DẪN CÀI ĐẶT LIGHTHOUSE BẰNG DOCKER (BEACON RPC VÀ SEPOLIA RPC)
---
##Yêu cầu
- Docker: Đã được cài đặt trên hệ thống.
- Docker Compose (tùy chọn): Để quản lý nhiều container.
- Máy chủ với ít nhất 4GB RAM, 2 CPU, và 100GB ổ cứng (khuyến nghị).
- Hệ điều hành Linux (Ubuntu 20.04 hoặc mới hơn được khuyến nghị).
---
## Bước 1: Cài đặt Docker
- Cài đặt Docker nếu chưa có:
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
- Kiểm tra phiên bản Docker:
```bash
docker --version
```
- Nên thấy phiên bản (ví dụ: Docker version 20.10.x).
- (Tùy chọn) Cài đặt Docker Compose:
```bash
sudo apt install -y docker-compose
```
- Kiểm tra:
```bash
docker-compose --version
```
- Thêm user vào nhóm Docker để chạy lệnh mà không cần sudo:
```bash
sudo usermod -aG docker $USER
newgrp docker
```
---
## Bước 2: Tạo thư mục làm việc
- Tạo thư mục để lưu trữ tệp cấu hình và dữ liệu:
```bash
mkdir -p $HOME/lighthouse-docker
cd $HOME/lighthouse-docker
```
---
## Bước 3: Tạo tệp JWT
- JWT là khóa xác thực để Geth và Lighthouse giao tiếp an toàn.
- Tạo tệp JWT:
```bash
mkdir -p $HOME/lighthouse-docker/jwt
openssl rand -hex 32 > $HOME/lighthouse-docker/jwt/jwt.hex
```
- Đặt quyền bảo mật:
```bash
chmod 600 $HOME/lighthouse-docker/jwt/jwt.hex
```
- Kiểm tra tệp:
```bash
cat $HOME/lighthouse-docker/jwt/jwt.hex
```
- Kết quả sẽ là một chuỗi hex, ví dụ:
```
d176xxxx....
```
---
## Bước 4: Tạo cấu hình Docker Compose
- Tạo tệp docker-compose.yml để chạy cả Geth và Lighthouse:
```bash
nano $HOME/lighthouse-docker/docker-compose.yml
```
- Dán nội dung sau vào tệp:
```yaml
version: "3.8"
services:
  geth:
    image: ethereum/client-go:stable
    container_name: geth
    volumes:
      - ./geth-data:/root/.ethereum
      - ./jwt:/jwt
    ports:
      - "8545:8545"
      - "8551:8551"
    command: >
      --sepolia
      --http
      --http.addr 0.0.0.0
      --http.api eth,net,web3
      --authrpc.port 8551
      --authrpc.jwtsecret /jwt/jwt.hex
      --syncmode snap
    networks:
      - lighthouse-network
    restart: unless-stopped

  lighthouse:
    image: sigp/lighthouse:stable
    container_name: lighthouse
    volumes:
      - ./lighthouse-data:/root/.lighthouse
      - ./jwt:/jwt
    ports:
      - "5052:5052"
      - "9000:9000/udp"
      - "9000:9000/tcp"
    command: >
      beacon_node
      --network sepolia
      --execution-endpoint http://geth:8551
      --execution-jwt /jwt/jwt.hex
      --http
      --http-address 0.0.0.0
      --http-port 5052
      --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io
    depends_on:
      - geth
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge
```
Lưu và thoát (Ctrl+O, Enter, Ctrl+X).
### Giải thích:
- geth: Sử dụng image chính thức của Geth (ethereum/client-go:stable).
- lighthouse: Sử dụng image chính thức của Lighthouse (sigp/lighthouse:stable).
- volumes: Gắn thư mục dữ liệu và tệp JWT vào container.
- ports: Mở cổng cần thiết (8545 cho Geth RPC, 8551 cho Geth auth, 5052 cho Lighthouse Beacon RPC).
- networks: Tạo mạng nội bộ để Geth và Lighthouse giao tiếp.
---
## Bước 5: Chạy Docker Compose
Khởi động các container:
```bash
cd $HOME/lighthouse-docker
docker-compose up -d
```
- Kiểm tra trạng thái container:
```bash
docker ps
```
- Nên thấy hai container geth và lighthouse đang chạy.
- Kiểm tra log nếu cần:
```bash
docker logs geth
docker logs lighthouse
```
---
## Bước 6: Kiểm tra Geth
- Kiểm tra cổng Geth:
```bash
netstat -tuln | grep 8545
```
nên thấy : 0.0.0.0:8545
```
netstat -tuln | grep 8551
```
nên thấy : 0.0.0.0:8551
- Kiểm tra RPC:
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545
```
- Kiểm tra trạng thái đồng bộ:
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
```
trả về "false" - đồng bộ hoàn tất.
---
## Bước 7: Kiểm tra Lighthouse
- Kiểm tra cổng Lighthouse:
```bash
netstat -tuln | grep 5052
```
- Nên thấy 0.0.0.0:5052.
- Kiểm tra Beacon RPC:
```bash
curl -X GET http://localhost:5052/eth/v1/node/version
```
- Nên trả về phiên bản Lighthouse (ví dụ: Lighthouse/v7.0.0).
---
## Bước 8: Thiết lập Firewall
- Cài đặt UFW:
```bash
sudo apt install -y ufw
```
- Mở các cổng cần thiết:
```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 8545
sudo ufw allow 8551
sudo ufw allow 5052
sudo ufw allow 9000/tcp
sudo ufw allow 9000/udp
sudo ufw enable
```
- Kiểm tra trạng thái:
```bash
sudo ufw status
```
## Bước 9: Quản lý và khắc phục sự cố
- Dừng container:
```bash
docker-compose down
```
- Khởi động lại:
```bash
docker-compose up -d
```
- Xóa dữ liệu và chạy lại (nếu cần):
```bash
docker-compose down
rm -rf $HOME/lighthouse-docker/geth-data $HOME/lighthouse-docker/lighthouse-data
docker-compose up -d
```
- Kiểm tra log nếu có lỗi:
```bash
docker logs geth
docker logs lighthouse
```
---
## Lưu ý
- Bảo mật: Đảm bảo tệp jwt.hex được bảo vệ và không chia sẻ công khai.
- Cập nhật: Để cập nhật Geth hoặc Lighthouse, kéo image mới:
```bash
docker pull ethereum/client-go:stable
docker pull sigp/lighthouse:stable
```
Sau đó chạy lại ``` docker-compose up -d ```
- Sao lưu: Sao lưu thư mục ``` $HOME/lighthouse-docker ``` để tránh mất dữ liệu.
---
# CHÚC CÁC BẠN THÀNH CÔNG
