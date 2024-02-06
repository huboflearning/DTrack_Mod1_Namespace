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

-NS1:-
1. sudo ip netns exec ns1 ip link set lo up
2. sudo ip netns exec ns1 ip link
   
-NS2:-
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

**Now set ip on ceth0 and ceth1 respectivly to ns1 and ns2**

-For NS1-
1. sudo ip netns exec ns1 ip addr add 172.16.1.10/24 dev ceth0
2. sudo ip netns ns1 ping -c 2 172.16.1.10 [ping own]
3. sudo ip netns exec ns1 ip route
4. sudo ip netns exec ns1 ping -c 172.16.1.1 [ping bridge network]

-For NS2-
1. sudo ip netns exec ns2 ip addr add 172.16.1.11/24 dev ceth1
2. sudo ip netns exec ns2 ping -c 2 172.16.1.11
3. sudo ip netns exec ns2 ip route
4. sudo ip netns exec ns2 ping -c 172.16.1.1

**Verify connecitvity between netns**
1. sudo nsenter --net=/var/run/netns/ns1
2. ping -c 2 172.16.1.11

3. sudo nsenter --net=/var/run/netns/ns2
4. ping -c 2 172.16.1.10
5. exit

***Now its time to connect to Internet from ns***

-First see that network is unreachable-
1. sudo ip netns exec ns1 ping -c 2 8.8.8.8

-Now check the routing table inside of ns1-
2. sudo ip netns exec route -n

- if we set default route this issue will be fixed-
1. sudo ip netns exec ns1 ip route add default via 172.16.1.1
2. sudo ip netns exec ns1 route -n

**Still 8.8.8.8 is unreachable from sn1 and ns2, neet to add ip forwardingin Host machine**

--terminal-1
--now trying to ping 8.8.8.8 again
sudo ip netns exec ns1 ping 8.8.8.8
--still unreachable

--terminal 2
--open tcpdump in eth0 to see the packet
sudo tcpdump -i eth0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
-->No Packet captured, let's capture traffic for br0
sudo tcpdump -i bro icmp
--> We can see the traffic at bro but we don't get response from eth0, it is 
Terminal 2:
becasue of ip forwarding issue.

-- lets forward the ip enabling ip forwarding by change value 0 to 1
1. sudo sysctl -w net.ipv4.ip_forward=1
2. sudo cat /proc/sys/net/ipv4/ip_forward
         1

--no packet captured, let's capture traffic for br0
sudo tcpdump -i br0 icmp

Terminal 2:
sudo tcpdump -i eth0 icmp
--> Now we can see packets at eth0

Still we cannot ping 8.8.8.8 from ns1.
To achive that we will need to do NAT at host machine by placing an iptables rule in the POSTROUTING chain of the nat table.

sudo iptables \
        -t nat \
        -A POSTROUTING \
        -s 172.16.1.0\24 ! -0 br0 \
        -j MASQUERADE

**Now we are getting ping respnse from 8.8.8.8**
sudo ip netns exec ns1 ping -c 2 8.8.8.8






