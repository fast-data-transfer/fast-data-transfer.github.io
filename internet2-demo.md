[[Home](index.md)]  [[Documentation](doc-fdt-ddcopy.md)]  [[Performance Tests](perf-disk-to-disk.md)] [Internet2 Demo]



### Internet2 2017 Technology Exchange: The Fast Data Transfer Tool


This tutorial took place on October 15, 2017 at the Internet2 2017 Technology exchange meeting in San Francisco. The use of virtual machines for the tutorial was made possible through the support of Google Cloud for Higher Education & Research.

We used the google cloud SDK (gcloud) for instantiating and customizing the virtual machines. The google cloud compute instance used for this tutorial is `n1-standard-4`. It has 15 GB of memory and 4 VCPUs.

Google currently provides with up to 2 Gbps per VCPU (hyperthread) capped at 16 Gbps per VM. We will test this in the tutorial. Google is looking for feedback from groups who would like to do really high throughput networking (40+ per VM). Feel free to let us know or the google team in this Internet2 meeting.


**Access to the Google Cloud virtual machines**

Open two windows and ssh into the servers using the username `fdt` and the IPs and password given at the tutorial.

```
ssh -l fdt <SERVER1-PUBLIC>
ssh -l fdt <SERVER2-PUBLIC>
```

On SERVER1:
```
export SERVER1=$(hostname --ip-address)
```

On SERVER2:
```
export SERVER2=$(hostname --ip-address)
```

Export these values on the other server too. These are the private IP addresses, not the ones you used above for login into the VMs.

_Note!_: If your session gets disconnected you will need to reset these shell variables or use the IPs directly in the examples below.


**Installation of Java and FDT**

On Both servers:
```
# install java and FDT
sudo yum install -y java-1.8.0-openjdk.x86_64
java -version

wget https://github.com/fast-data-transfer/fdt/releases/download/0.25.1/fdt.jar
java -jar fdt.jar -version
```


**Getting help**

The `-help` flag will print out all flags with descriptions.
```
java -jar fdt.jar -help
java -jar fdt.jar -help >& help.txt
less help.txt
```

**Copy a file**

Send one file called "local.data" from the local system directory to another computer in the "/tmp" folder, with default parameters

First the FDT server needs to be started on the "remote" system (SERVER2). The default settings will be used, which implies the default port, 54321, on both the client and the server. The `-S` flag is used to disable the standalone mode. When using this flag, the server will stop after the copy session will finish.

On SERVER2:
```
java -jar fdt.jar -S
```

Then the client will be started on the "local" system (SERVER1) specifying the sourcefile, the remote address (or hostname) where the server was started in the previous step and the destination directory.

On SERVER1:
```
for i in `seq 100`; do echo "$i local data"; done > local.data  # dummy file
java -jar fdt.jar -c $SERVER2 -d /tmp ./local.data              # the transfer
```

_Secure Copy (SCP) Mode_

In this mode the server will be started on the remote systemautomatically by the local FDT client using SSH.

On SERVER1:
```
java -jar fdt.jar ./local.data fdt@$SERVER2:/home/fdt/
```

If the remoteuser parameter is not specified the local user, running the fdt command, will be used to login on the remote system.

**Recursive copying**

To get the content of an entire folder and all its children, located in the home directory of the user, we will use the `-r` (recursive mode) flag. Furthermore, the `-pull` flag will be used to pull the data from the server. In the client-server mode, the acces to the server will be restricted to some IP addresses only. This is done with the `-f` flag.

Multiple IP addresses may be specified using the -f flag using ':' separator. If the IP address of the client is not specified in the allowed IP addresses, the connection will be closed. In the following command the server is started in standalone mode, which means that will continue to run after the session will finish. The transfer rate for every client session can be limited on the server with the `--limit` flag (``--limit 4M`` for 4Mbps)

On SERVER2:
```
java -jar fdt.jar -f $SERVER1:$SERVER2
```


The command for the local client will be.

On SERVER1
```
java -jar fdt.jar -pull -r -c $SERVER2 -d ./share /usr/share  
```

_Recursive copying with more read threads_

```
# SERVER2
java -jar fdt.jar -f $SERVER1:$SERVER2 -rCount 100 -wCount 100
# SERVER1
java -jar fdt.jar -pull -r -rCount 100 -c $SERVER2 -d ./share /usr/share
```

P.S. there is a bug on writing a directory with a subdirectory structure with multiple threads.

_Recursive copying with more streams_

```
# SERVER2
java -jar fdt.jar -f $SERVER1:$SERVER2 -rCount 100 -wCount 100 -P 10
# SERVER1
java -jar fdt.jar -pull -r -rCount 100 -P10 -c $SERVER2 -d ./share /usr/share
```

P.S. you can run out of memory when using many threads and streams. Usually the error message from FDT/Java is pretty obvious in this case.


_Recursive copying in SCP mode_

In this mode only the order of the parameters will be changed, and `-r` is the only argument that must be added (`-pull` is implicit). The same authentication policies apply.

On SERVER1:
```
java -jar fdt.jar -r  fdt@$SERVER2:/usr/share ./share
```

**Transfer with list of files**

The user can define a list of files (one filename per lin ) to be transferred. FDT will detect if the files are located on multiple devices and will use a dedicated thread for each device.

On SERVER2
```
java -jar fdt.jar -S
```

ON SERVER1
```
java -jar fdt.jar -fl ./file_list.txt -c <remote_address> -d /home/fdt/files
```


**Testing network connectivity**

To test the network connectivity one can start a transfer of data from /dev/zero on the server (SERVER1) to /dev/null on the client (SERVER2) using 10 streams in blocking mode, for both the server and the client with 8 MBytes buffers. The server will stop after the test is finished.

On the SERVER2 (server):
```
java -jar fdt.jar -bio -bs 8M -f $SERVER1 -S
```

On SERVER1 (client):
```
java -jar fdt.jar -c $SERVER2 -bio -P 10 -d /dev/null /dev/zero
```

 _SCP mode_

On VM1:
```
[local computer]$ java -jar fdt.jar -bio -P 10 /dev/zero $SERVER2:/dev/null
```


**Testing local read/write performance**

To test the local read/write performance of the local disk the
DDCopy may be used.

- The following command will copy the entire partition
/dev/dsk/c0d1p1 to /dev/null reporting every 2 seconds ( the default )
the I/O speed

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy if=/dev/dsk/c0d1p1 of=/dev/null
```

- To test the write speed of the file system using a 1GB file
read from /dev/zero the following command may be used. The operating
system will sync() the data to the disk. The data will be read/write
using 10MB buffers

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/user/1GBTestFile bs=10M count=100 flags=NOSYNC
```

OR

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/user/1GBTestFile bs=1M bn=10 count=100 flags=NOSYNC
```

**Launching FDT as Agent**

```
java -jar fdt.jar -tp <transfer,ports,separated,by,comma> -p <portNo> -agent
```

- Sending coordinator message to the agent:

```
java -jar fdt.jar -dIP <destination-ip> -dp <destination-port> -sIP <source-ip> -p <source-port> -d /tmp/destination/files -fl /tmp/file-list-on-source.txt -coord
```
- Retrieving session log file. 

To retrieve session log file user needs to provide at least these parameters:

```
java -jar fdt.jar  -c <source-host> -d /tmp/destination/files -sID <session-ID>
```

- To retrieve list of files on custom path there is a custom mode which can be used.

```
java -jar fdt.jar  -c <source-host> -ls /tmp/
```

