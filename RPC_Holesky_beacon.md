```bash
mkdir $HOME/holesky
```
Bước 3: Creat JWT file
```bash
mkdir -p $HOME/holesky/jwt
openssl rand -hex 32 > $HOME/holesky/jwt/jwt-holesky.hex
```
Set law.
```bash
chmod 600 $HOME/holesky/jwt/jwt-holesky.hex
# Kiểm tra tệp:
cat $HOME/holesky/jwt/jwt-holesky.hex
# Kết quả sẽ là một chuỗi hex, ví dụ:
# d176xxxx....
```
```bash
cd $HOME/holesky
nano docker-compose.yml
```
```yml
cat docker-compose.yml
version: '3.8'
services:
  geth-holesky:
    image: ethereum/client-go:stable
    tty: true
    container_name: geth-holesky
    volumes:
      - ./geth-holesky-data:/root/.ethereum
      - ./jwt/jwt-holesky.hex:/jwt-holesky.hex
    ports:
      - "8700:8700"      # HTTP RPC
      - "8701:8701"      # Auth RPC
      - "8702:8702/tcp" # P2P TCP
      - "8702:8702/udp" # P2P UDP
    command: >
      --holesky
      --http
      --http.addr 0.0.0.0
      --http.port 8700
      --http.api eth,net,web3
      --http.corsdomain "*"
      --http.vhosts "*"
      --authrpc.addr 0.0.0.0
      --authrpc.port 8701
      --authrpc.jwtsecret /jwt-holesky.hex
      --authrpc.vhosts "*"
      --syncmode snap
      --port 8702
    networks:
      - lighthouse-network
    restart: unless-stopped

  lighthouse-holesky:
    image: sigp/lighthouse:v7.1.0
    tty: true
    container_name: lighthouse-holesky
    depends_on:
      - geth-holesky
    volumes:
      - ./lighthouse-holesky-data:/root/.lighthouse
      - ./jwt/jwt-holesky.hex:/jwt-holesky.hex
    ports:
      - "8704:8704"      # HTTP API
      - "8705:8705/tcp"  # P2P TCP
      - "8705:8705/udp"  # P2P UDP
    command: sh -c "sleep 10 && lighthouse beacon --network holesky --execution-endpoint http://geth-holesky:8701 --execution-jwt /jwt-holesky.hex --http --http-address 0.0.0.0 --http-port 8704 --checkpoint-sync-url https://checkpoint-sync.holesky.ethpandaops.io --port 8705"
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge

```
