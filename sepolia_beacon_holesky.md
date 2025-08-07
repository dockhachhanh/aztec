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
    image: sigp/lighthouse:v7.0.1
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
    ports:
      - "8546:8546"
    command: >
      --holesky
      --http
      --http.addr 0.0.0.0
      --http.api eth,net,web3
      --http.corsdomain "*"
      --http.vhosts "*"
      --syncmode snap
    networks:
      - lighthouse-network
    restart: unless-stopped

networks:
  lighthouse-network:
    driver: bridge
```
