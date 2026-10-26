# Connect namespaces.

## 🧪 Lab Goal

**Create:**

2 network namespaces
Connect them using a veth pair
Assign IPs
Ping between them ✅

**⚙️ Step 1 — Create Namespaces**

```
sudo ip netns add ns1
sudo ip netns add ns2
```

**🔗 Step 2 — Create veth Pair**

`sudo ip link add veth-ns1 type veth peer name veth-ns2`

**🔌 Step 3 — Attach Each End to a Namespace**

```
sudo ip link set veth-ns1 netns ns1
sudo ip link set veth-ns2 netns ns2
```

**🌐 Step 4 — Assign IP Addresses**

```
sudo ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth-ns1
sudo ip netns exec ns2 ip addr add 10.0.0.2/24 dev veth-ns2
```

**🟢 Step 5 — Bring Interfaces UP**

```
sudo ip netns exec ns1 ip link set veth-ns1 up
sudo ip netns exec ns1 ip link set lo up

sudo ip netns exec ns2 ip link set veth-ns2 up
sudo ip netns exec ns2 ip link set lo up
```

**🧪 Step 6 — Test Connectivity**

`sudo ip netns exec ns1 ping 10.0.0.2`
If everything is correct:
`64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=...`

## Connect This to Docker

`Container <-- veth --> docker0 bridge <-- veth --> Container`

**Step 7 — Challenge:**

From ns1 try:
`curl 10.0.0.2`
It won’t work yet — no service running

**Some Deep Understanding:**

Why do we need lo up?
What happens if subnet is different?
Why does ping fail if interface is down?

**1. Why do we need lo up?**

lo = loopback interface, IP = 127.0.0.1 which is used for internal communication inside the namespace.
When you create a namespace, the lo interface is down by default.
Many things depend on loopback:
- internal service communication
- DNS resolution
- apps binding to 127.0.0.1
- system tools expecting loopback to be up

**2. What happens if subnet is different?**

ns1 - 10.0.0.2/24
ns2 - 192.168.1.2/24
PING -> FAIL, Why? Devices only talk directly if they are in the same subnet.

**3. Why does ping fail if interface is down?**

What does “interface down” mean?
`ip link set veth1 down`
Equivalent to : unplugging a network cable
What happens:
- no packet transmission
- no ARP requests
- no ICMP (ping)

**Ping requires:**

Interface UP ✅
IP assigned ✅
Route available ✅

What actually happens under the hood? When you ping:
System checks routing table
Finds outgoing interface
Sends packet through interface

**If interface is DOWN:**

`No interface → No packet → No ping`

DevOps-Level Insight
Whenever debugging networking (Docker/K8s):
Always check in this order:

```
1. Interface UP?
2. IP assigned?
3. Route exists?
4. Subnet correct?
5. NAT / firewall rules?
```

**🔍 1. Check: Interface UP?**

```
sudo ip netns exec ns1 ip link
```
✅ What You Should See
`veth-c1: <BROADCAST,MULTICAST,UP,LOWER_UP>`
UP + LOWER_UP = GOOD

**FIX** - `sudo ip netns exec ns1 ip link set veth-c1 up`

**🌐 2. Check: IP Assigned?**

```
sudo ip netns exec ns1 ip a
```
Good Output: `inet 172.18.0.2/16`
IP is assigned.

**🧭 3. Check: Route Exists?**

```
sudo ip netns exec ns1 ip route
```
**Good Output:**
```
default via 172.18.0.1
172.18.0.0/16 dev veth-c1 proto kernel scope link
```

**FIX** - `sudo ip netns exec ns1 ip route add default via 172.18.0.1`

**🌍 4. Check: Subnet Correct?**

All devices should be in same network range.
Example:
ns1 - 172.18.0.2/16
ns2 - 172.18.0.3/16
bridge - 172.18.0.1/16

**🔥 5. Check: NAT / Firewall Rules**

`sudo iptables -t nat -L -n -v`

**Good Output:**

`MASQUERADE  all  --  172.18.0.0/16  anywhere`

**FIX** - `sudo iptables -t nat -A POSTROUTING -s 172.18.0.0/16 -o eth0 -j MASQUERADE`

**Check forwarding rules:**

`sudo iptables -L FORWARD -n -v`

**Good Output**: `ACCEPT tcp -- anywhere 172.18.0.2 tcp dpt:80`

**FIX** - `sudo iptables -A FORWARD -p tcp -d 172.18.0.2 --dport 80 -j ACCEPT`

**🧪 REAL DEBUG WALKTHROUGH (How You Should Think)**

**If ping fails:**

**Step 1:**

ip netns exec ns1 ip link

👉 Interface UP?

**Step 2:**

ip netns exec ns1 ip a

👉 IP present?

**Step 3:**

ip netns exec ns1 ip route

👉 Default route?

**Step 4:**

ip netns exec ns1 ping 172.18.0.1

👉 Can reach gateway?

**Step 5:**

iptables -t nat -L

👉 NAT exists?

**🧠 Pro DevOps Insight (This is GOLD)**

**Whenever networking fails:**

Layer 1 → Interface
Layer 2 → IP
Layer 3 → Routing
Layer 4 → NAT
Layer 5 → Firewall

👉 90% issues are solved by this flow
















