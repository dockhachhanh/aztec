- Find Ip
```bash
docker ps
```
```bash
docker inspect <aztec-node-container-id> | grep IPAddress
```
- change <aztec-node-container-id> = container ID
- you recieve a IP: 172.17.0.X
```bash
curl -X POST -H "Content-Type: application/json" \
--data '{"method":"nodeAdmin_rollbackTo","params":[31020]}' \
http://172.17.0.X:8880
```
- You can see: {}
- Okie wait and checking logs
- patient.
