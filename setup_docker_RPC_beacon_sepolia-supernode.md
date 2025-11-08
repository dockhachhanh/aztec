install follow https://github.com/dockhachhanh/aztec/blob/main/setup_docker_RPC_beacon_sepolia.md before

```bash
nano docker-compose.yml
```
```bash
version: '3.8'

services:
  geth:
    image: ethereum/client-go:v1.16.7
    container_name: geth
    tty: true
    volumes:
      - ./geth-data:/root/.ethereum
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "8545:8545"
      - "8551:8551"
    command: >
      --sepolia
      --http
      --http.addr 0.0.0.0
      --http.api eth,net,web3
      --authrpc.addr 0.0.0.0
      --authrpc.port 8551
      --authrpc.jwtsecret /jwt.hex
      --authrpc.vhosts=*
      --ws
      --syncmode snap
    networks:
      - lighthouse-network
    restart: unless-stopped

  lighthouse:
    image: sigp/lighthouse:v8.0.0-rc.1
    container_name: lighthouse
    tty: true
    depends_on:
      - geth
    volumes:
      - ./lighthouse-data:/root/.lighthouse
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "5052:5052"
      - "9000:9000/udp"
      - "9000:9000/tcp"
    command: >
      sh -c "sleep 10 && lighthouse beacon \
      --network sepolia \
      --supernode \
      --execution-endpoint http://geth:8551 \
      --execution-jwt /jwt.hex \
      --http \
      --http-address 0.0.0.0 \
      --http-port 5052 \
      --metrics \
      --metrics-address 0.0.0.0 \
      --metrics-allow-origin '*' \
      --builder https://builder-relay-sepolia.flashbots.net \
      --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io"
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge


```
