# Topic 5 - Docker networking

## I. Baseline network inspection (Default bridge)

### 1. Show existing networks
```
docker network ls
```

### 2. Inspect default bridge
```
docker network inspect bridge
```
### 3. Run two containers on default bridge and test connectivity
```
docker run -d --name n1 alpine sleep 2000
docker run -d --name n2 alpine sleep 2000
docker exec n1 ip a
n2_ip=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' n2)
sudo docker exec n1 ping -c2 ${n2_ip}
```

## 4. Remove containers
```
docker rm -f n1 n2
```

## II. User-defined bridge
### 1. Create network
```
docker network create lab-net
```
### 2. Verify
```
docker network inspect lab-net
```
### 3. Launch two containers on new network
```
docker run -d --name web1 --network lab-net nginx:alpine
docker run -d --name busy1 --network lab-net alpine sleep 2000
```
### 4. Test DNS and traffic
```
docker exec busy1 ping -c2 web1
docker exec busy1 wget -qO- http://web1
```
### 5. Disconnect and reconnect
```
docker network disconnect lab-net busy1
docker network connect lab-net busy1
```
### 6. Cleanup
```
docker rm -f web1 busy1
docker network rm lab-net
```

## III. Host network
### 1. Run nginx with host network
```
docker run -d --name hostnetweb --network host nginx:alpine
```
### 2. Verify port listening
Run on host shell
```
ss -lntp
```

### 3. Access via host IP on port 80

### 4. Cleanup
```
docker rm -f hostnetweb
```

## IV. None network
### 1. Run isolated container
```
docker run -d --name none1 --network none alpine sleep 2000
```
### 2. Validate absence of network
```
docker exec none1 ip a
docker exec none1 ping -c2 8.8.8.8
## should fail
```
### 3. Cleanup
```
docker rm -f none1
```
