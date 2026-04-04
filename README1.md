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

<img width="1148" height="480" alt="image" src="https://github.com/user-attachments/assets/435ac239-6508-4d97-b7fa-ab789dfc7ee8" />

Inside Container:
`docker exec -it test sh`
`ip addr`

<img width="860" height="237" alt="image" src="https://github.com/user-attachments/assets/98240e57-3707-4071-8bb6-46efc87eb244" />


### Use nsenter
From host:
`sudo nsenter -t <PID> -n ip a`
eg. `sudo nsenter -t 12345 -n ip a`

<img width="922" height="238" alt="image" src="https://github.com/user-attachments/assets/2b83f45f-e150-4bbb-bc62-4c96af26a1be" />

### See Namespace Links
`ls -l /proc/<PID>/ns`

<img width="788" height="262" alt="image" src="https://github.com/user-attachments/assets/ca447dc0-c359-4393-bef8-bd80d792c349" />

### Compare with Host Namespace
`ls -l /proc/1/ns/net`

<img width="802" height="245" alt="image" src="https://github.com/user-attachments/assets/fda719b0-68ff-4c40-b8ab-df876c24280b" />

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
