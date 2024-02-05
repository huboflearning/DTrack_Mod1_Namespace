# DTrack_Mod1_Namespace

In this Repository, I will add steps details about 
1. How to create Namespace,
2. assign interface in namespace,
3. create Virtual Ethernet (veth) and communicate between namespaces, and
4. Doing NAT at Host and establishing communication Namesspace to Internet.
5. **Please check Namespace command below:**

**Step 0: List all interfaces in the Host:**
1. sudo ip link
Find Routing Table:
2. sudo route -n

**Step- 1: Create two network namespace**
1. sudo ip netns add ns1
2. sudo ip netns add ns2

list the created network namespaces
3. sudo ip netns list
List network namespaces from the location:
4. sudo ls /var/run/netns

**Step- 2: By default, network interfaces of created netns are down inlcuding loopback interface. make them up:**

**NS1:**
1. sudo ip netns exec ns1 ip link set lo up
2. sudo ip netns exec ns1 ip link
   
**NS2:**
1. sudo ip netns exec ns2 ip link set lo up
2. sudo ip netns exec ns2 ip link

**Create a bridge netowork on the host that will connect ns1 and ns2 with host**
1. sudo ip link add br0 type bridge
2. sudo ip link set br0 up [up the bridge network]
3. sudo ip link [check the status whether it is up]

 **Configure ip to the bridge network**
 1. sudo ip addr add 172.16.1.1/24 dev br0
 2. sudo ip addr
 3. sudo ping -c 2 172.16.1.1 [testing own whether it is pingable]

**Create two veth interfaces for two netns to connect to bridg network br0**

1. sudo ip link add veth0 type veth peer name ceth0 [for ns1]
2. sudo ip link add veth1 type veth peer name ceth1 [for ns2]

**Connect both veth with bridge network**
1. sudo ip link set veth0 master bro
2. sudp ip link set veth0 up

1. sudo ip link set veth1 master bro
2. sudo ip link set veth1 up

**Connect other end of veth to NS1 and NS2**

-NS1-
1. sudo ip link set ceth0 netns ns1 [connect ceth0 to NS1]
2. sudp ip netns exec ns1 ip link set ceth0 up [up ceth0]
3. sudo ip link [show interface in host]
4. sudo ip netns exec ns1 ip link [show interface in NS1]

-NS2-
1. sudo ip link set ceth1 netns ns2 [connect ceth1 to NS2]
2. sudp ip netns exec ns2 ip link set ceth1 up [up ceth1]
3. sudo ip link [show interfcae in host]
4. sudo ip netns exec ns1 ip link [show interfce in NS2]


