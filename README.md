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


