```bash
nano docker-compose.yml
```
```bash
version: "3.9"

services:
  geth:
    image: ethereum/client-go:v1.16.4
    container_name: geth
    tty: true
    user: "root"
    volumes:
      - ./geth-data:/home/quy/geth-sepolia
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "8645:8645"   # HTTP
      - "8651:8651"   # AuthRPC
      - "30304:30304" # P2P
      - "8661:8661"   # Metrics
    command: >
      --sepolia
      --syncmode snap
      --cache=61440
      --maxpeers=100
      --datadir /home/quy/geth-sepolia
      --http
      --http.addr 0.0.0.0
      --http.port 8645
      --http.api eth,net,web3,engine,txpool
      --authrpc.addr 0.0.0.0
      --authrpc.port 8651
      --authrpc.jwtsecret /jwt.hex
      --authrpc.vhosts=*
      --http.corsdomain=*
      --metrics
      --metrics.port 8661
      --pprof
      --port 30304
      --blobpool.datadir /home/quy/geth-sepolia/blobs
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
      - ./lighthouse-data:/home/quy/.lighthouse
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "5052:5052"
      - "9000:9000/udp"
      - "9000:9000/tcp"
    command: >
      sh -c "
        sleep 10 &&
        lighthouse beacon_node --supernode
        --network sepolia
        --execution-endpoint http://geth:8651
        --execution-jwt /jwt.hex
        --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io
        --http
        --http-address 0.0.0.0
        --http-port 5052
        --metrics
        --metrics-address 0.0.0.0
        --metrics-allow-origin '*'
        --suggested-fee-recipient 0x7b896407539587864401F92535cFa2b2f86bfaeD
        --builder https://builder-relay-sepolia.flashbots.net
      "
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge

```
