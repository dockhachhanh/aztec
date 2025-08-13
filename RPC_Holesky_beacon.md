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
      - "8700:8546"      # HTTP RPC
      - "8701:8552"      # Auth RPC
      - "8702:30303/tcp" # P2P TCP
      - "8703:30303/udp" # P2P UDP
    command: >
      --holesky
      --http
      --http.addr 0.0.0.0
      --http.port 8546
      --http.api eth,net,web3
      --http.corsdomain "*"
      --http.vhosts "*"
      --authrpc.addr 0.0.0.0
      --authrpc.port 8552
      --authrpc.jwtsecret /jwt-holesky.hex
      --authrpc.vhosts "*"
      --syncmode snap
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
      - "8704:5054"      # HTTP API
      - "8705:9002/tcp"  # P2P TCP
      - "8706:9002/udp"  # P2P UDP
    command: sh -c "sleep 10 && lighthouse beacon --network holesky --execution-endpoint http://geth-holesky:8552 --execution-jwt /jwt-holesky.hex --http --http-address 0.0.0.0 --http-port 5054 --checkpoint-sync-url https://checkpoint-sync.holesky.ethpandaops.io"
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge

```
