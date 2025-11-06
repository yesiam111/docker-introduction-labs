# Topic 7 - Docker storage

## I. Container Lifecycle and Ephemeral Storage

### 1. Run a container
```
docker run -it --name temp-container ubuntu bash
```

### 2. Inside container
```
echo "persistent test" > /tmp/test.txt
exit
```

### 3. Verify container removal effect
```
docker rm temp-container
docker run -it --name temp-container2 ubuntu bash
cat /tmp/test.txt
## file not found
```

## II. Using Docker Volumes
### 1. Create a volume
```
docker volume create mydata
```

### 2. Run container with volume mount
```
docker run -it --name volume-test -v mydata:/data ubuntu bash
```

### 3. Inside container
```
echo "saved to volume" > /data/file.txt
exit
```

### 4. Remove container
```
docker rm -f volume-test
```

### 5. Run new container with same volume
```
docker run -it -v mydata:/data ubuntu cat /data/file.txt
```

## III. Using Bind Mounts
### 1. Create a directory on host
```
mkdir /tmp/hostdata
echo "bind mount demo" > /tmp/hostdata/host.txt
```

### 2. Run container with bind mount
```
docker run -it --name bind-test -v /tmp/hostdata:/mnt ubuntu bash
```

### 3. Inside container
```
cat /mnt/host.txt
echo "added from container" >> /mnt/host.txt
exit
```

### 4. Check on host:
```
cat /tmp/hostdata/host.txt
```

### 5. Cleanup
```
docker rm -f bind-test
```

## IV. Using tmpfs Mounts
### 1. Run container with tmpfs:
```
docker run -it --name tmpfs-test --tmpfs /tmp/data:rw,size=64m ubuntu bash
```

### 2. Inside container:
```
echo "temporary cache" > /tmp/data/cache.txt
exit
```

### 3. Restart container:
```
docker start -ai tmpfs-test
## run inside container
cat /tmp/data/cache.txt
```
