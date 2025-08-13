```bash
rm -rf jwt/jwt-holesky.hex
mkdir -p jwt
echo -n "0x$(openssl rand -hex 32)" > jwt/jwt-holesky.hex
```

```yaml
version: '3.8'
services:
  geth:
    image: ethereum/client-go:stable
    tty: true
    container_name: geth
    volumes:
      - ./geth-data:/root/.ethereum
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "8545:8545"
      - "8551:8551"
      - "30303:30303/tcp"
      - "30303:30303/udp"
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
    image: sigp/lighthouse:v7.1.0
    tty: true
    container_name: lighthouse
    depends_on:
      - geth
    volumes:
      - ./lighthouse-data:/root/.lighthouse
      - ./jwt/jwt.hex:/jwt.hex
    ports:
      - "5052:5052"
      - "9000:9000/udp"
      - "9000:9000/tcp"
    command: sh -c "sleep 10 && lighthouse beacon --network sepolia --execution-endpoint http://geth:8551 --execution-jwt /jwt.hex --http --http-address 0.0.0.0 --http-port 5052 --checkpoint-sync-url https://beaconstate-sepolia.chainsafe.io"
    networks:
      - lighthouse-network
    restart: unless-stopped

  geth-holesky:
    image: ethereum/client-go:stable
    tty: true
    container_name: geth-holesky
    volumes:
      - ./geth-holesky-data:/root/.ethereum
      - ./jwt/jwt-holesky.hex:/jwt-holesky.hex
    ports:
      - "8546:8546"
      - "8552:8552"
      - "30304:30303/tcp"
      - "30304:30303/udp"
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
      - "5054:5054"
      - "9002:9002/udp"
      - "9002:9002/tcp"
      - "8707:5055"      # Metrics (optional)
    command: sh -c "sleep 10 && lighthouse beacon --network holesky --execution-endpoint http://geth-holesky:8552 --execution-jwt /jwt-holesky.hex --http --http-address 0.0.0.0 --http-port 5054 --checkpoint-sync-url https://checkpoint-sync.holesky.ethpandaops.io --port 9004 --metrics --metrics-address 0.0.0.0 --metrics-port 5055"
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge

```
