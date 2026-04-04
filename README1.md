# Network Namespace

## What is a Network Namespace?
A network namespace is like a completely separate network stack:
- own IP address
- own routing table
- own interfaces (eth0,lo,etc.)
- own firewall rules

Note: Docker creates a new network namespace for each container.

### Create an EC2 Ubuntu instance & install Docker using provided install_docker.sh file.

### Run a nginx container:
`docker run -d --name test nginx:alpine`

Verify running container: 
`docker ps`

<img width="1027" height="367" alt="image" src="https://github.com/user-attachments/assets/62f52319-df96-4f37-801d-957146a5ba5f" />

### Find Container PID
`docker inspect -f '{{.State.Pid}}' test`

<img width="912" height="60" alt="image" src="https://github.com/user-attachments/assets/0149533d-7ded-4448-99e4-fcc56caf271f" />


### Check Network Namespace (Host vs Container)
On Host (EC2):
`ip addr or ip a`

Inside Container:
`docker exec -it test sh`
`ip addr`

### Use nsenter
From host:
`sudo nsenter -t <PID> -n ip a`
eg. `sudo nsenter -t 12345 -n ip a`

### See Namespace Links
`ls -l /proc/<PID>/ns`

### Compare with Host Namespace
`ls -l /proc/1/ns/net`

Control Flow:
```
[ EC2 Host Namespace ]
        |
    docker0 (bridge)
        |
    veth pair
        |
[ Container Namespace ]
    eth0 → 172.17.x.x
```
