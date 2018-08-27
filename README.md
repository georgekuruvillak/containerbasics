# Container Basics
Lets go through the Linux namespaces initially.

## Mount Namespaces
In a terminal session, try creating a mount namespace.
```
mkdir /tmp/mount_ns
unshare -m /bin/bash
readlink /proc/$$/ns/mnt
mount -n -t tmpfs tmpfs /tmp/mount_ns
df -h | grep mount_ns
cat /proc/mounts | grep mount_ns
```
Open another terminal session and check if you can still see the mounted volume.
```
readlink /proc/$$/ns/mnt
cat /proc/mounts | grep mount_ns
df -h | grep mount_ns
```
We see that in the root namespace we are not able to see the mounted volume that we created in the mount namespace we created.

## UTS Namespace
In a terminal session, try creating a UTS namespace.
```
hostname
unshare -u /bin/bash
hostname UTS_Namespace
cat /proc/sys/kernel/hostname
```
Open another terminal session and check if the hostname of the machine has changed globally.

## PID Namespace
In a terminal session, try creating a PID namespace.
```
unshare --pid /bin/bash
```
Oh man!! What happened?
```
unshare -fp /bin/bash
```
I see all the processes in the PID namespace. Thats bad. Lets rectify it.
```
unshare -fp --mount-proc /bin/bash
```
Voila!!!

## User Namespace
In a terminal session, see which user you are logged in as.
```
whoami
```
Now create a user namespace
```
unshare --user /bin/bash
whoami
```
Why am I 'Nobody'? Let's change that.
```
unshare --user --map-root-user /bin/bash
whoami
cat /proc/$$/uid_map
```
Now I am root inside but non-root user outside. What if I don't want to be root inside?
Exit out of the user namespace and create another one.
```
unshare --user /bin/bash
echo $$
```
Open another terminal session. And change the uid_map.
```
id
newuidmap [pid] [uid_to_map_in_namespace] [uid_in_parent_namespace] 1
```
In the user namespace, lets see the mapping now.
```
whoami
cat /proc/$$/uid_map
```

## Network namespace

Check the interfaces in the root namespace and the addresses assigned.
```
ip link
ip addr show
```
Create two namespaces ns1 and ns2.
```
ip netns add ns1
ip netns add ns2
ip netns
```
See the interfaces in each namespace.
```
ip netns exec ns1 ip link
ip netns exec ns2 ip link
```
Now lets make them communicate with each other.
We need a virtual bridge to connect the namespaces
```
apt-get install -y openvswitch-switch
```
Create a virtual bridge in the root namespace.
```
ovs-vsctl add-br OVS-1
ovs-vsctl show
ip a s OVS-1
```
Next let us create virtual eth pipes.
```
ip link add eth1-ns1 type veth peer name veth-ns1
ip link add eth1-ns2 type veth peer name veth-ns2
```
List out the interfaces.
```
ip link
```
Assigning one end of the eth pipes to respective namespaces.
```
ip link set eth1-ns1 netns ns1
ip link set eth1-ns2 netns ns2
```
Check the interfaces in the namespaces.
```
ip netns exec ns1 ip link
ip netns exec ns2 ip link
```
Check interfaces in the root namespace.
```
ip link
```
Connect the other end of the two virtual pipes, the veth-ns1 and veth-ns2.
```
ovs-vsctl add-port OVS-1 veth-ns1
ovs-vsctl add-port OVS-1 veth-ns2
ovs-vsctl show
```
The last thing left to do is bring the network interfaces up and assign IP addresses:
```  
ip link set veth-ns1 up
ip link set veth-ns2 up
ip netns exec ns1 ip link set dev lo up
ip netns exec ns1 ip link set dev eth1-ns1 up
ip netns exec ns1 ip address add 192.168.0.1/24 dev eth1-ns1
ip netns exec ns1 ip a s

ip netns exec ns2 ip link set dev lo up
ip netns exec ns2 ip link set dev eth1-ns2 up
ip netns exec ns2 ip address add 192.168.0.2/24 dev eth1-ns2
ip netns exec ns2 ip a s
```
Try communicating from ns1 to ns2
```
ip netns exec ns1 ping -c 3 192.168.0.2
```
Cool!!!
