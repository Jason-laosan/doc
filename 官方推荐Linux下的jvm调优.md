### Startup Files

The following parameters must be added to the /etc/rc.d/rc.local file that gets executed during system startup.

```
<-- begin
#max file count updated ~256 descriptors per 4Mb. 
Specify number of file descriptors based on the amount of system RAM.
echo "6553"> /proc/sys/fs/file-max
#inode-max 3-4 times the file-max
#file not present!!!!!
#echo"262144"> /proc/sys/fs/inode-max
#make more local ports available
echo 1024 25000> /proc/sys/net/ipv4/ip_local_port_range
#increase the memory available with socket buffers
echo 2621143> /proc/sys/net/core/rmem_max
echo 262143> /proc/sys/net/core/rmem_default
#above configuration for 2.4.X kernels
echo 4096 131072 262143> /proc/sys/net/ipv4/tcp_rmem
echo 4096 13107262143> /proc/sys/net/ipv4/tcp_wmem
#disable "RFC2018 TCP Selective Acknowledgements," and 
"RFC1323 TCP timestamps" echo 0> /proc/sys/net/ipv4/tcp_sack
echo 0> /proc/sys/net/ipv4/tcp_timestamps
#double maximum amount of memory allocated to shm at runtime
echo "67108864"> /proc/sys/kernel/shmmax
#improve virtual memory VM subsystem of the Linux
echo "100 1200 128 512 15 5000 500 1884 2"> /proc/sys/vm/bdflush
#we also do a sysctl
sysctl -p /etc/sysctl.conf
-- end -->
```

Additionally, create an /etc/sysctl.conf file and append it with the following values:

```
<-- begin
 #Disables packet forwarding
net.ipv4.ip_forward = 0
#Enables source route verification
net.ipv4.conf.default.rp_filter = 1
#Disables the magic-sysrq key
kernel.sysrq = 0
fs.file-max=65536
vm.bdflush = 100 1200 128 512 15 5000 500 1884 2
net.ipv4.ip_local_port_range = 1024 65000
net.core.rmem_max= 262143
net.core.rmem_default = 262143
net.ipv4.tcp_rmem = 4096 131072 262143
net.ipv4.tcp_wmem = 4096 131072 262143
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
kernel.shmmax = 67108864
```

### File Descriptors

You may need to increase the number of file descriptors from the default. Having a higher number of file descriptors ensures that the server can open sockets under high load and not abort requests coming in from clients.

Start by checking system limits for file descriptors with this command:

```
cat /proc/sys/fs/file-max
8192
```

The current limit shown is 8192. To increase it to 65535, use the following command (as root):

```
echo "65535"> /proc/sys/fs/file-max
```

To make this value to survive a system reboot, add it to **/etc/sysctl.conf** and specify the maximum number of open files permitted:

```
fs.file-max = 65535
```

Note that the parameter is not **proc.sys.fs.file-max**, as one might expect.

To list the available parameters that can be modified using **sysctl**:

```
sysctl -a
```

To load new values from the **sysctl.conf** file:

```
sysctl -p /etc/sysctl.conf
```

To check and modify limits per shell, use the following command:

limit
The output will look something like this:

```
cputime         unlimited
filesize        unlimited
datasize        unlimited
stacksize       8192 kbytes
coredumpsize    0 kbytes
memoryuse       unlimited
descriptors     1024
memorylocked    unlimited
maxproc         8146
openfiles       1024
```

The openfiles and descriptors show a limit of 1024. To increase the limit to 65535 for all users, edit **/etc/security/limits.conf** as root, and modify or add the nofile setting (number of file) entries:

- ```
      soft    nofile                     65535
  ```

- ```
      hard    nofile                     65535
  ```

The character "*****" is a wildcard that identifies all users. You could also specify a user ID instead.

Then edit **/etc/pam.d/login** and add the line:

```
session required /lib/security/pam_limits.so
```

On Red Hat, you also need to edit **/etc/pam.d/sshd** and add the following line:

```
session required /lib/security/pam_limits.so
```

On many systems, this procedure will be sufficient. Log in as a regular user and try it before doing the remaining steps. The remaining steps might not be required, depending on how pluggable authentication modules (PAM) and secure shell (SSH) are configured.

### Virtual Memory

To change virtual memory settings, add the following to **/etc/rc.local**:

```
echo 100 1200 128 512 15 5000 500 1884 2> /proc/sys/vm/bdflush
```

For more information, view the man pages for `bdflush`.

### Network Interface

To ensure that the network interface is operating in full duplex mode, add the following entry into **/etc/rc.local**:

```
mii-tool -F 100baseTx-FD eth0
```

where eth0 is the name of the network interface card (NIC).

### Disk I/O Settings

To tune disk I/O performance for non SCSI disks
Test the disk speed.

1. Use this command:

```
/sbin/hdparm -t /dev/hdX
```

1. Enable direct memory access (DMA).

Use this command:

```
/sbin/hdparm -d1 /dev/hdX
```

1. Check the speed again using the `hdparm` command.

Given that DMA is not enabled by default, the transfer rate might have improved considerably. In order to do this at every reboot, add the `/sbin/hdparm -d1 /dev/hdX` line to `/etc/conf.d/local.start`, `/etc/init.d/rc.local`, or whatever the startup script is called.

For information on SCSI disks, see: System Tuning for Linux Servers — SCSIOpens a new window.

TCP/IP Settings

To tune the TCP/IP settings
Add the following entry to /etc/rc.local

```
echo 30> /proc/sys/net/ipv4/tcp_fin_timeout
echo 60000> /proc/sys/net/ipv4/tcp_keepalive_time
echo 15000> /proc/sys/net/ipv4/tcp_keepalive_intvl
echo 0> /proc/sys/net/ipv4/tcp_window_scaling
```

Add the following to `/etc/sysctl.conf`

```
# Disables packet forwarding
net.ipv4.ip_forward = 0
# Enables source route verification
net.ipv4.conf.default.rp_filter = 1
# Disables the magic-sysrq key
kernel.sysrq = 0
net.ipv4.ip_local_port_range = 1204 65000
net.core.rmem_max = 262140
net.core.rmem_default = 262140
net.ipv4.tcp_rmem = 4096 131072 262140
net.ipv4.tcp_wmem = 4096 131072 262140
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_keepalive_time = 60000
net.ipv4.tcp_keepalive_intvl = 15000
net.ipv4.tcp_fin_timeout = 30
Add the following as the last entry in /etc/rc.local
```

`sysctl -p /etc/sysctl.conf`
Reboot the system.

Use this command to increase the size of the transmit buffer:

```
tcp_recv_hiwat ndd /dev/tcp 8129 32768
```

### UDP Buffer Sizes

GlassFish Server uses User Datagram Protocol (UDP) for the transmission of multicast messages to GlassFish Server instances in a cluster. For peak performance from a GlassFish Server cluster that uses UDP multicast, limit the need to retransmit UDP messages. To limit the need to retransmit UDP messages, set the size of the UDP buffer to avoid excessive UDP datagram loss.

To Determine an Optimal UDP Buffer Size
The size of UDP buffer that is required to prevent excessive UDP datagram loss depends on many factors, such as:

- The number of instances in the cluster
- The number of instances on each host
- The number of processors
- The amount of memory
- The speed of the hard disk for virtual memory

If only one instance is running on each host in your cluster, the default UDP buffer size should suffice. If several instances are running on each host, determine whether the UDP buffer is large enough by testing for the loss of UDP packets.

Note:

On Linux systems, the default UDP buffer size might be insufficient even if only one instance is running on each host. In this situation, set the UDP buffer size as explained in To Set the UDP Buffer Size on Linux Systems.

1. Ensure that no GlassFish Server clusters are running.

If necessary, stop any running clusters as explained in "To Stop a Cluster" in Oracle GlassFish Server High Availability Administration Guide.

1. Determine the absolute number of lost UDP packets when no clusters are running.

How you determine the number of lost packets depends on the operating system. For example:

On Linux systems, use the netstat -su command and look for the packet receive errors count in the Udp section.

On AIX systems, use the netstat -s command and look for the fragments dropped (dup or out of space) count in the ip section.

1. Start all the clusters that are configured for your installation of GlassFish Server.

Start each cluster as explained in "To Start a Cluster" in Oracle GlassFish Server High Availability Administration Guide.
\4. Determine the absolute number of lost UDP packets after the clusters are started.

1. If the difference in the number of lost packets is significant, increase the size of the UDP buffer.

#### To Set the UDP Buffer Size on Linux Systems

On Linux systems, a default UDP buffer size is set for the client, but not for the server. Therefore, on Linux systems, the UDP buffer size might have to be increased. Setting the UDP buffer size involves setting the following kernel parameters:

```
net.core.rmem_max

net.core.wmem_max

net.core.rmem_default

net.core.wmem_default
```

Set the kernel parameters in the `/etc/sysctl.conf` file or at runtime.

If you set the parameters in the `/etc/sysctl.conf` file, the settings are preserved when the system is rebooted. If you set the parameters at runtime, the settings are not preserved when the system is rebooted.

To set the parameters in the `/etc/sysctl.conf` file, add or edit the following lines in the file:

```
net.core.rmem_max=rmem-max
net.core.wmem_max=wmem-max
net.core.rmem_default=rmem-default
net.core.wmem_default=wmem-default
```

To set the parameters at runtime, use the sysctl command.

```
$ /sbin/sysctl -w net.core.rmem_max=rmem-max \
net.core.wmem_max=wmem-max \
net.core.rmem_default=rmem-default \
net.core.wmem_default=wmem-default
```

***Example 5-1 Setting the UDP Buffer Size in the `/etc/sysctl.conf` File***

This example shows the lines in the `/etc/sysctl.conf` file for setting the kernel parameters for controlling the UDP buffer size to 524288.

```
net.core.rmem_max=524288
net.core.wmem_max=524288
net.core.rmem_default=524288
net.core.wmem_default=524288
```

***Example 5-2 Setting the UDP Buffer Size at Runtime***

This example sets the kernel parameters for controlling the UDP buffer size to 524288 at runtime.

```
$ /sbin/sysctl -w net.core.rmem_max=524288 \
net.core.wmem_max=52428 \
net.core.rmem_default=52428 \
net.core.wmem_default=524288
net.core.rmem_max = 524288
net.core.wmem_max = 52428
net.core.rmem_default = 52428
net.core.wmem_default = 524288
```