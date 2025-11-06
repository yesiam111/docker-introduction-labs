# Topic 4 - Docker swarm

## I. Initialize docker swarm cluster
### 1. On manager node
```
docker swarm init --advertise-addr <manager-IP>
```
### 2. Note the join token output
### 3. On worker node
```
docker swarm join --token <token> <manager-IP>:2377
```
### 4. Verify
```
docker node ls
```
## II. Create and Manage Services
### 1. Create a service:
```
docker service create --name web --replicas 3 -p 8080:80 nginx
```
### 2. Check service and tasks
```
docker service ls
docker service ps web
```

### 3. Access via browser
```
http://<manager-IP>:8080
```
## III. Scale Service
### 1. Scale to 5 replicas
```
docker service scale web=5
```
### 2. Observe scheduling
```
docker service ps web
```

### 3. Scale down to 2 replicas
```
docker service scale web=2
```
## III. Deploy Stack Using Compose
### 1. Create `stack.yml`
```
version: "3"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  redis:
    image: redis:alpine
```

### 2. Deploy
```
docker stack deploy -c stack.yml mystack
```

### 3. List stack and services:
```
docker stack ls
docker stack services mystack
```
## IV. Overlay Network and Service Communication
### 1. Create an overlay network
```
docker network create -d overlay appnet
```

### 2. Deploy services on that network
```
docker service create --name db --network appnet redis
docker service create --name api --network appnet nginx
```

### 3. Verify connectivity
```
docker exec -it $(docker ps -qf "name=api") ping db
```
## V. Cleanup
### 1. List active stacks and services
```
docker stack ls
docker service ls
```

### 2. Remove all stacks
Repeat for each stack name if multiple exist
```
docker stack rm mystack
```
### 3. Remove standalone services
Command removes all running services
```
docker service rm $(docker service ls -q)
```
### 4. Remove overlay networks
Deletes all overlay networks created for Swarm
```
docker network rm $(docker network ls -q -f "driver=overlay")
```
### 5. Leave Swarm mode
Use --force only on the manager node
```
docker swarm leave --force
```
### 6. Verify cleanup
```
docker info | grep Swarm
docker ps -a
docker network ls
```
