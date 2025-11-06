# Topic 02 - Docker image and registry

## I. Single stage image build
### 1. Create single state dockerfile & build image
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
### 2. Create `Dockerfile`
```
FROM golang:1.22-alpine
WORKDIR /app
COPY main.go .
RUN go mod init example.com/goapp
RUN go build -o server .
EXPOSE 8080
ENTRYPOINT ["./server"]
```
### 3. Build docker image
```
sudo docker build -t goapp-single:1.0 .
```
### 4. Check docker image
```
sudo docker image ls
```
- Sample output:
```
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
goapp-single           1.0       0a65c856a6e7   58 seconds ago   303MB
```
### 5. Run container app
```
sudo docker run -d -p 18080:8080 --name goapp-single goapp-single:1.0
```
### 6. Check for running container and try to send request to container
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

### 7. Cleanup
```
docker rm -f goapp-single
```
## II. Multi stage container build
### 1. Create single state dockerfile & build image
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
### 2.Create `Dockerfile`
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
### 3. Build docker image
```
sudo docker build -t goapp-multi:1.0 .
```
### 4. Check docker image
```
sudo docker image ls
```
- Sample output:
```
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
goapp-multi            1.0       ebe73e2f9283   2 seconds ago    14.8MB
```
### 5. Run container app
```
sudo docker run -d -p 18081:8080 --name goapp-multi goapp-multi:1.0
```
### 6.Check for running container and try to send request to container
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
### 7. Cleanup
```
docker rm -f goapp-multi
```
## III. Docker hub image management
### 1. Create account on docker hub
### 2. Login at CLI with OTP
```
sudo docker login
```
or using username/password
```
sudo docker login -u <username>
```
### 3. Create image repo on docker hub <>
- Refer: https://docs.docker.com/docker-hub/repos/create/
- Create a repo named `goapp-multi`
### 4. Tag built image
```
sudo docker tag goapp-multi:1.0 <account>/goapp-multi:1.0
```
### 5. Push image to container repo
```
sudo docker push <account>/goapp-multi:1.0
```
- Sample output:
```
The push refers to repository [docker.io/<account>/goapp-multi]
0080306a8751: Pushed 
fca748f05368: Pushed 
abfcb263a588: Pushed 
1.0: digest: sha256:d85667f05c1218716991ca2d91f1c97ed8fe9fd6cb60c3132b29c72ea00d4d2a size: 945
```





