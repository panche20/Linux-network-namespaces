# Connect namespaces.

🧪 Lab Goal

Create:
2 network namespaces
Connect them using a veth pair
Assign IPs
Ping between them ✅

⚙️ Step 1 — Create Namespaces
```
sudo ip netns add ns1
sudo ip netns add ns2
```

🔗 Step 2 — Create veth Pair
`sudo ip link add veth-ns1 type veth peer name veth-ns2`

🔌 Step 3 — Attach Each End to a Namespace
```
sudo ip link set veth-ns1 netns ns1
sudo ip link set veth-ns2 netns ns2
```

🌐 Step 4 — Assign IP Addresses
```
sudo ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth-ns1
sudo ip netns exec ns2 ip addr add 10.0.0.2/24 dev veth-ns2
```

🟢 Step 5 — Bring Interfaces UP
```
sudo ip netns exec ns1 ip link set veth-ns1 up
sudo ip netns exec ns1 ip link set lo up

sudo ip netns exec ns2 ip link set veth-ns2 up
sudo ip netns exec ns2 ip link set lo up
```

🧪 Step 6 — Test Connectivity
`sudo ip netns exec ns1 ping 10.0.0.2`
If everything is correct:
`64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=...`

## Connect This to Docker

`Container <-- veth --> docker0 bridge <-- veth --> Container`

Step 7 — Challenge:
From ns1 try:
`curl 10.0.0.2`
It won’t work yet — no service running

Some Deep Understanding:
Why do we need lo up?
What happens if subnet is different?
Why does ping fail if interface is down?

1. Why do we need lo up?
lo = loopback interface, IP = 127.0.0.1 which is used for internal communication inside the namespace.
When you create a namespace, the lo interface is down by default.
Many things depend on loopback:
- internal service communication
- DNS resolution
- apps binding to 127.0.0.1
- system tools expecting loopback to be up

2. What happens if subnet is different?
ns1 - 10.0.0.2/24
ns2 - 192.168.1.2/24
PING -> FAIL, Why?














