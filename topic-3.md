# Topic 3 - Docker compose
## I. Basic docker compose deployment
### 1. Create `docker-compose.yml` file
```
services:
  web:
    image: nginx
    ports:
      - "8085:80"
    networks:
      - appnet
  redis:
    image: redis
    networks:
      - appnet
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - appnet
volumes:
  dbdata:
networks:
  appnet:
```

### 2. Deploy docker compose stack
```
sudo docker compose up -d
```
### 3. Check docker compose container status
```
sudo docker compose ps
```
- Sample output:
```
NAME              IMAGE     COMMAND                  SERVICE   CREATED          STATUS          PORTS
compose-db-1      mysql     "docker-entrypoint.s…"   db        33 seconds ago   Up 33 seconds   3306/tcp, 33060/tcp
compose-redis-1   redis     "docker-entrypoint.s…"   redis     33 seconds ago   Up 33 seconds   6379/tcp
compose-web-1     nginx     "/docker-entrypoint.…"   web       33 seconds ago   Up 33 seconds   0.0.0.0:8085->80/tcp, [::]:8085->80/tcp
```
### 4. Check docker compose container logs
```
sudo docker compose logs
```
- Sample output:
```
web-1  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
web-1  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
web-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
web-1  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf

...

db-1   | 2025-11-04 18:32:04+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.5.0-1.el9 started.
db-1   | 2025-11-04 18:32:04+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
db-1   | 2025-11-04 18:32:04+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.5.0-1.el9 started.
db-1   | 2025-11-04 18:32:04+00:00 [Note] [Entrypoint]: Initializing database files
...

redis-1  | Starting Redis Server
redis-1  | 1:C 04 Nov 2025 18:32:04.517 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis-1  | 1:C 04 Nov 2025 18:32:04.517 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
```

### 5. Check connection between compose's container
Execute curl from `web` container to `redis` container on port 6379.
```
sudo docker compose exec -t web curl -vvv redis:6379
```
- Sample output:
```
18:37:50.458115 [0-x] == Info: [READ] client_reset, clear readers
18:37:50.458613 [0-0] == Info: Host redis:6379 was resolved.
18:37:50.458902 [0-0] == Info: IPv6: (none)
18:37:50.459113 [0-0] == Info: IPv4: 172.18.0.2
18:37:50.459300 [0-0] == Info: [SETUP] added
18:37:50.459670 [0-0] == Info:   Trying 172.18.0.2:6379...
...
```
### 6. Cleanup
```
sudo docker compose down
```
- Sample output:
```
[+] Running 4/4
 ✔ Container compose-redis-1  Removed                                                                                                                                                                                                                                  0.2s 
 ✔ Container compose-db-1     Removed                                                                                                                                                                                                                                  0.6s 
 ✔ Container compose-web-1    Removed                                                                                                                                                                                                                                  0.2s 
 ✔ Network compose_appnet     Removed   
```

## II. Docker compose deployment with Dockerfile and env file
### 1. Prepare source code, `Dockerfile` and `.env` file
- Source code: `main.go`
```
package main
import (
  "fmt"
  "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello from Docker!")
}
func main() {
  http.HandleFunc("/", handler)
  http.ListenAndServe(":8080", nil)
}
```
- `Dockerfile` file
```
# Stage 1: build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY main.go .
RUN go mod init example.com/goapp
RUN go build -o server .

# Stage 2: minimal runtime
FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/server .
EXPOSE 8080
ENTRYPOINT ["./server"]
```
- `.env` file
```
APP_PORT=19080
```
### 2. Prepare `docker-compose.yml` file
- `docker-compose.yml`
```
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    env_file:
      - .env
    ports:
      - "${APP_PORT}:8080"
    restart: unless-stopped
    networks:
      - appnet
  redis:
    image: redis
    networks:
      - appnet
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - appnet
volumes:
  dbdata:
networks:
  appnet:
```
### 3. Run and build docker container at deployment
```
docker compose up --build
```
### 4. Check for container status, logs
Reference to I.3, I.4
```
sudo docker compose ps
sudo docker compose logs
```
### 5. Cleanup
```
sudo docker compose down
```
