# Topic 02 - Docker image and registry

## 1. Create single state dockerfile & build image
Prepare directory and sample source code
```
mkdir single_stage_build
cd single_stage_build
cat > main.go<<EOF
package main
import (
  "fmt"
  "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello from Docker single stage build!")
}
func main() {
  http.HandleFunc("/", handler)
  http.ListenAndServe(":8080", nil)
}
EOF
```
Create `Dockerfile`
```
FROM golang:1.22-alpine
WORKDIR /app
COPY main.go .
RUN go mod init example.com/goapp
RUN go build -o server .
EXPOSE 8080
ENTRYPOINT ["./server"]
```
Build docker image
```
sudo docker build -t goapp-single:1.0 .
```
Check docker image
```
sudo docker image ls
```
- Sample output:
```
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
goapp-single           1.0       0a65c856a6e7   58 seconds ago   303MB
```
Run app
```
sudo docker run -d -p 18080:8080 --name goapp-single goapp-single:1.0
```
Check for running container and try to send request to container
```
sudo docker ps
curl http://localhost:18080
```
- Sample output:
```
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                           NAMES
6676d70a44b0   goapp-single:1.0   "./server"               6 seconds ago    Up 5 seconds    0.0.0.0:18080->8080/tcp, [::]:18080->8080/tcp   goapp-single

...

Hello from Docker single stage build!
```

## 2. Create single state dockerfile & build image
Prepare directory and sample source code
```
mkdir multi_stage_build
cd multi_stage_build
cat > main.go<<EOF
package main
import (
  "fmt"
  "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello from Docker multi stage build!")
}
func main() {
  http.HandleFunc("/", handler)
  http.ListenAndServe(":8080", nil)
}
EOF
```
Create `Dockerfile`
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
Build docker image
```
sudo docker build -t goapp-multi:1.0 .
```
Check docker image
```
sudo docker image ls
```
- Sample output:
```
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
goapp-multi            1.0       ebe73e2f9283   2 seconds ago    14.8MB
```
Run app
```
sudo docker run -d -p 18081:8080 --name goapp-multi goapp-multi:1.0
```
Check for running container and try to send request to container
```
sudo docker ps
curl http://localhost:18081
```
- Sample output:
```
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                           NAMES
47c81735fce4   goapp-multi:1.0    "./server"               12 seconds ago   Up 11 seconds   0.0.0.0:18081->8080/tcp, [::]:18081->8080/tcp   goapp-multi

...

Hello from Docker multi stage build!
```

