# HƯỚNG DẪN CHI TIẾT CHẠY AZTEC PROVER NODE
## Mục tiêu
Triển khai Aztec Prover Node trên mạng alpha-testnet (Sepolia, chain ID 11155111), tích hợp với:
Execution Layer (EL): Geth node cung cấp JSON-RPC.

Consensus Layer (CL): Lighthouse node cung cấp blob sidecars.

MinIO: Lưu trữ blob sidecars.

Blob Proxy: API phục vụ blob sidecars từ MinIO.

Prover Node: Đồng bộ L2 block, lấy proving job, và publish proof lên L1 (Sepolia).

Yêu cầu
Máy chủ:
Hệ điều hành: Ubuntu 22.04 (hoặc tương tự).

CPU: 4 cores trở lên.

RAM: 16GB trở lên.

Ổ cứng: 500GB SSD (cho Geth, Lighthouse, MinIO, và dữ liệu node).

Kết nối mạng ổn định, IP công khai (ví dụ: 113.161.16.182).

Phần mềm:
Docker, Docker Compose.

Node.js (v18+), npm.

MinIO client (mc).

Ví Ethereum:
Private key và address (ví dụ: 0x889FF238475cAA63eFf42C595a052bcF1eBd9226) với ít nhất 1 ETH trên Sepolia.

Cổng mở:
EL: 8645 (Geth).

CL: 5052 (Lighthouse).

MinIO: 9002 (API), 9003 (console).

Blob Proxy: 8081.

Prover Node: 8080 (HTTP), 40400 (TCP/UDP cho P2P).

Các bước triển khai
Bước 1: Chuẩn bị môi trường
Cập nhật hệ thống:

sudo apt update
sudo apt upgrade -y

Cài đặt Docker và Docker Compose:
```bash
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
```
  - Đăng xuất và đăng nhập lại để áp dụng nhóm docker.

  - Kiểm tra phiên bản:
```
docker --version
docker-compose --version
```
- Cài đặt Node.js và npm:
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node --version
npm --version
```
- Cài đặt MinIO client (mc):
```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
mc --version
```
- Tạo thư mục làm việc:
```bash
mkdir -p $HOME/aztec-blob-storage
cd $HOME/aztec-blob-storage
```
- Mở cổng firewall:
```bash
sudo ufw allow 8645/tcp
sudo ufw allow 5052/tcp
sudo ufw allow 9002/tcp
sudo ufw allow 9003/tcp
sudo ufw allow 8081/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 40400/tcp
sudo ufw allow 40400/udp
sudo ufw enable
sudo ufw status
```
## Bước 4: Thiết lập MinIO để lưu trữ Blob Sidecars
- Chạy MinIO với Docker:
```bash
docker run -d -p 9002:9000 -p 9003:9001 --name minio -e "MINIO_ROOT_USER=admin" -e "MINIO_ROOT_PASSWORD=minio123456" minio/minio server /data --console-address ":9001"
```
- Cấu hình MinIO client:
```
mc alias set minio http://127.0.0.1:9002 admin minio123456
mc mb minio/aztec-blobs
mc ls minio/aztec-blobs
```
# Bước 5: Thu thập Blob Sidecars với fetch-blobs.js
Tạo fetch-blobs.js:
```bash
nano /home/quy/aztec-blob-storage/fetch-blobs.js
```
- Dán nội dung:
```javascript
const axios = require('axios');
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

const s3Client = new S3Client({
  endpoint: 'http://113.161.16.182:9002',
  region: 'us-east-1',
  credentials: { accessKeyId: 'admin', secretAccessKey: 'minio123456' },
  forcePathStyle: true
});

async function fetchAndStoreBlobs() {
  try {
    const response = await axios.get('http://113.161.16.182:5052/eth/v1/beacon/blob_sidecars/head');
    const sidecars = response.data.data;
    for (const sidecar of sidecars) {
      const blobData = {
        blob: sidecar.blob,
        kzg_commitment: sidecar.kzg_commitment,
        kzg_proof: sidecar.kzg_proof
      };
      const key = `${sidecar.kzg_commitment}.json`;
      await s3Client.send(new PutObjectCommand({
        Bucket: 'aztec-blobs',
        Key: key,
        Body: JSON.stringify(blobData),
        ContentType: 'application/json'
      }));
      console.log(`Stored blob ${key}`);
    }
  } catch (error) {
    console.error('Error:', error.message);
  }
}

setInterval(fetchAndStoreBlobs, 12000);
fetchAndStoreBlobs();
```
- Lưu và thoát (Ctrl+O, Enter, Ctrl+X).
- Cài đặt dependencies:
```bash
cd $HOME/aztec-blob-storage
npm install axios @aws-sdk/client-s3
```
- Chạy fetch-blobs.js:
```bash
node fetch-blobs.js > fetch-blobs.log 2>&1 &
```
- Kiểm tra bucket:
```bash
mc ls minio/aztec-blobs
mc cat minio/aztec-blobs/<some_kzg_commitment>.json | head -n 10
```
- Dự kiến thấy file JSON với blob, kzg_commitment, kzg_proof.
# Bước 6: Thiết lập Blob Proxy với blob-proxy.js
- Tạo blob-proxy.js:
```bash
nano /home/quy/aztec-blob-storage/blob-proxy.js
```
- Dán nội dung:
```javascript
const express = require('express');
const { S3Client, GetObjectCommand } = require('@aws-sdk/client-s3');
const app = express();
const port = 8081;

const s3Client = new S3Client({
  endpoint: 'http://113.161.16.182:9002',
  region: 'us-east-1',
  credentials: { accessKeyId: 'admin', secretAccessKey: 'minio123456' },
  forcePathStyle: true
});

app.get('/blob/:key', async (req, res) => {
  const key = req.params.key;
  try {
    const command = new GetObjectCommand({
      Bucket: 'aztec-blobs',
      Key: `${key}.json`
    });
    const { Body } = await s3Client.send(command);
    const data = await streamToString(Body);
    res.setHeader('Content-Type', 'application/json');
    res.send(data);
  } catch (error) {
    console.error(`Lỗi khi lấy blob ${key}:`, error.message);
    res.status(404).send({ error: `Blob ${key} không tìm thấy` });
  }
});

async function streamToString(stream) {
  const chunks = [];
  for await (const chunk of stream) {
    chunks.push(chunk);
  }
  return Buffer.concat(chunks).toString('utf-8');
}

app.listen(port, () => {
  console.log(`Blob proxy chạy trên cổng ${port}`);
});
```
- Lưu và thoát.
- Cài đặt dependencies:
```bash
npm install express @aws-sdk/client-s3
```
- Chạy blob-proxy.js:
```bash
node --trace-warnings blob-proxy.js > blob-proxy.log 2>&1 &
```
- Kiểm tra proxy:
```bash
curl http://113.161.16.182:8081/blob/0x8090e6877f72db669901ca0492ea5bcc06a8e72a0e08f22b802dde95360764e91f0b2a3319d7f02c3f89a4c973cc0cbe
cat blob-proxy.log
```
## Triển khai Aztec Prover Node
```bash
nano $HOME/aztec-blob-storage/.env
```
- Dán nội dung:
```
ETHEREUM_HOSTS=http://192.168.1.103:8645
L1_CONSENSUS_HOST_URLS=http://113.161.16.182:5052
BLOB_STORAGE_URL=http://113.161.16.182:8081/blob
PROVER_PUBLISHER_PRIVATE_KEY=YOUR_PRIVATE_KEY
PROVER_ID=YOUR_ADDRESS
P2P_ENABLED=true
```
- Thay YOUR_PRIVATE_KEY bằng private key (ví dụ: 0x1234...).

- Thay YOUR_ADDRESS bằng address (ví dụ: 0x889FF238...).

- Lưu và thoát.

- Tạo file docker-compose.yml:
```bash
nano $HOME/aztec-blob-storage/docker-compose.yml
```
- Dán nội dung:
```yaml
name: aztec-prover
services:
  prover-node:
    tty: true
    image: aztecprotocol/aztec:alpha-testnet
    command:
      - node
      - --no-warnings
      - /usr/src/yarn-project/aztec/dest/bin/index.js
      - start
      - --prover-node
      - --archiver
      - --network
      - alpha-testnet
    depends_on:
      broker:
        condition: service_started
        required: true
    environment:
      DATA_DIRECTORY: /data
      DATA_STORE_MAP_SIZE_KB: "134217728"
      ETHEREUM_HOSTS: "${ETHEREUM_HOSTS}"
      L1_CONSENSUS_HOST_URLS: "${L1_CONSENSUS_HOST_URLS}"
      BLOB_STORAGE_URL: "${BLOB_STORAGE_URL}"
      LOG_LEVEL: debug
      PROVER_BROKER_HOST: http://broker:8080
      PROVER_PUBLISHER_PRIVATE_KEY: "${PROVER_PUBLISHER_PRIVATE_KEY}"
      P2P_ENABLED: "true"
      P2P_ANNOUNCE_ADDRESSES: "/ip4/113.161.16.182/tcp/40400"
    ports:
      - "8080:8080"
      - "40400:40400"
      - "40400:40400/udp"
    volumes:
      - /home/quy/aztec-blob-storage/prover-data:/data
    restart: unless-stopped

  agent:
    tty: true
    image: aztecprotocol/aztec:alpha-testnet
    command:
      - node
      - --no-warnings
      - /usr/src/yarn-project/aztec/dest/bin/index.js
      - start
      - --prover-agent
      - --network
      - alpha-testnet
    environment:
      PROVER_AGENT_COUNT: "1"
      PROVER_AGENT_POLL_INTERVAL_MS: "10000"
      PROVER_BROKER_HOST: http://broker:8080
      PROVER_ID: "${PROVER_ID}"
    pull_policy: always
    restart: unless-stopped

  broker:
    tty: true
    image: aztecprotocol/aztec:alpha-testnet
    command:
      - node
      - --no-warnings
      - /usr/src/yarn-project/aztec/dest/bin/index.js
      - start
      - --prover-broker
      - --network
      - alpha-testnet
    environment:
      DATA_DIRECTORY: /data
      ETHEREUM_HOSTS: "${ETHEREUM_HOSTS}"
      LOG_LEVEL: debug
    volumes:
      - /home/quy/aztec-blob-storage/broker-data:/data
    restart: unless-stopped
```
- Lưu ý:
    - Thay /home/quy bằng /home/tung nếu cần.
    - P2P_ANNOUNCE_ADDRESSES sử dụng IP công khai Public IP.
- Chạy Prover Node:
```bash
cd /home/quy/aztec-blob-storage
docker-compose up -d
```

