# Topic 6 - Container security

## I. Namespace Isolation Demonstration

### 1. Run two containers:
```
docker run -dit --name n1 alpine
docker run -dit --name n2 alpine
```

### 2. Inside each container:
```
docker exec n1 ps aux
docker exec n2 ps aux
```

Observe that each has its own PID tree (PID 1 = sh or ash).

### 3. Check network namespace isolation:
```
docker exec n1 ip a
docker exec n2 ip a
```

Each container should have its own network interface (eth0).

### 4. Check hostname isolation:
```
docker exec n1 hostname
docker exec n2 hostname
```

### 5. Cleanup
```
docker rm -f n1 n2
```
## II. Control Groups (cgroups) Resource Limiting

### 1. Run a container limited to 1 CPU core, 128 MB RAM:
```
docker run -dit --name limitdemo --cpus="1.0" --memory="128m" alpine
```

### 2. Inside container, stress system:
```
docker exec -it limitdemo sh
apk add --no-cache stress-ng
stress-ng --cpu 2 --vm 2 --vm-bytes 256M --timeout 30s
```

Observe container throttling or OOM termination.
```
tail /var/log/syslog -f
```

** Install rsyslog
```
sudo apt install rsyslog -y
sudo systemctl enable --now rsyslog
```

### 3. View cgroup info from host:
```
cid=$(docker inspect -f '{{ .Id }}' limitdemo)
ls /sys/fs/cgroup/system.slice/docker-${cid}.scope/
cat /sys/fs/cgroup/system.slice/docker-${cid}.scope/memory.max
cat /sys/fs/cgroup/system.slice/docker-${cid}.scope/cpu.max
```

### 4. Cleanup
```
docker rm -f limitdemo
```

## III. Least Privilege and Capability Hardening

### 1. Run a root container and try privileged operation
```
docker run -it --name privtest alpine sh
whoami
mknod /dev/test c 1 5
```

### 2. Drop all capabilities:
```
docker run -it --rm --cap-drop ALL alpine sh
id
```

- Try privileged operations:
```
mknod /dev/test-a c 1 5
```

It should fail.

### 3. Run with minimal capability needed:
```
docker run -it --cap-drop ALL --cap-add MKNOD alpine sh
mknod /dev/test-b c 1 5
```

Works only if capability explicitly added.

### 4. Cleanup
```
docker rm -f privtest
```

## IV. Image Vulnerability Scanning with Trivy
### 1. Install Trivy:
```
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
```

### 2. Scan official Nginx image:
```
trivy image nginx:latest
```

### 3. Interpret results:

- High/Critical vulnerabilities → must update base image.

- Automate in CI/CD:
  ```
  trivy image --exit-code 1 --severity HIGH,CRITICAL goapp-multi:1.0
  ```
