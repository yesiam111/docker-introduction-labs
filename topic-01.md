# Topic 1 - Docker introduction lab

## 1. Install docker engine

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable docker --now
```

### Verify

```bash
sudo systemctl status docker
sudo docker info
```

- Sample output:
```bash
docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since .....
...
Client: Docker Engine - Community
 Version: ...
...
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
...
```

## 2. Pull and list docker image
Pull docker image
```
sudo docker pull nginx
```
List docker image
```
sudo docker image ls
```
- Sample output:
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d261fd19cb63   13 hours ago   152MB
```

## 3. Run and list docker container
Run a container
```
sudo docker run nginx
```
Run a container in detached mode
```
sudo docker run -d nginx
```
Run a `named` container in detached mode
```
sudo docker run -d --name sample_cont nginx
```
List running container
```
sudo docker ps
```
- Sample output:
```
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
249a81b39f3b   nginx     "/docker-entrypoint.…"   4 seconds ago        Up 4 seconds        80/tcp    sample_cont
2752a4709b93   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    happy_beaver
```

## 4. Execute shell in container
Execuate shell in container
```
sudo docker exec -it sample_cont /bin/bash
```
Run command inside container, then exit
```
ls
cat docker-entrypoint.sh
exit
```

## 5. Stop and delete container
Stop container
```
sudo docker stop sample_cont
```
List **all** container
```
sudo docker ps -a
```
- Sample output:
```
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS                      PORTS     NAMES
249a81b39f3b   nginx         "/docker-entrypoint.…"   5 minutes ago   Exited (0) 33 seconds ago             sample_cont
2752a4709b93   nginx         "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes                80/tcp    happy_beaver
```
Delete stopped container
```
sudo docker rm sample_cont
```
Confirm container deleted
```
sudo docker ps -a | grep sample_cont
```



