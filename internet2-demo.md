[[Home](index.md)]  [[Documentation](doc-fdt-ddcopy.md)]  [[Performance Tests](perf-disk-to-disk.md)] [Internet2 Demo]


### Internet2 2017 Technology Exchange: The Fast Data Transfer Tool


Tutorial given on October 15, 2017 at the Internet2 2017 Technology exchange meeting in San Francisco. The use of virtual machines for the tutorial was made possible through the support of Google Cloud for Higher Education & Research.

The link to the agenda: [indico.hep.caltech.edu/indico/event/fdt](http://indico.hep.caltech.edu/indico/event/fdt)

**FDT Pointers**

Github:
- [github.com/fast-data-transfer](https://github.com/fast-data-transfer)

Email:
- support-fdt@monalisa.cern.ch

Twitter:
- @fastdt

Developer Team:
- Justas Balcas, jbalcas@caltech.edu
- Raimondas Sirvinskas, raimondas.sirvinskas@cern.ch, [www.linkedin.com/in/rsirvins](www.linkedin.com/in/rsirvins)


**Setup**

We used the google cloud SDK (gcloud) for instantiating and customizing the virtual machines. The google cloud compute instance used for this tutorial is `n1-standard-4`. It has 15 GB of memory and 4 VCPUs. We used `--image-family=centos-7` and  `--image-project=centos-cloud`.

Google currently provides network connectivity of up to 2 Gbps per VCPU (hyperthread) with maximum 16 Gbps per VM. We will test this in the tutorial for our instances. Google is looking for feedback from groups who would like to do really high throughput networking (40+ per VM). If you are interested, feel free to let us know or the google team at this Internet2 meeting.

If you are running this tutorial on your own, you can use any 2 servers or virtual machines with Centos 7. Other linux distributions work too, of course. But some of the command syntax might be different, for example the installation of java.

_Google Cloud Internet2 Egress Waiver_

For members of Internet2 in the Higher Education category, GC offers a waiver for data egress fees. You can find more details and fill out the waiver form here:
- [https://support.google.com/cloud/answer/7476636?hl=en](https://support.google.com/cloud/answer/7476636?hl=en)

**Access to the Google Cloud virtual machines**

Open two windows and ssh into the servers using the username `fdt` and the IPs and password given at the tutorial.

```
ssh -l fdt <SERVER1-PUBLIC>
ssh -l fdt <SERVER2-PUBLIC>
```

You can check the instance capability using any standard linux tools, for example `htop`.

Save the private IP of the servers in shell variables and export on both servers.

On SERVER1:
```
export SERVER1=$(hostname --ip-address)
export SERVER2=<server2 ip from below>
```

On SERVER2:
```
export SERVER2=$(hostname --ip-address)
export SERVER1=<server1 ip from below>
```

Export these values on the other server too. These are the private IP addresses, not the ones you used above for login into the VMs.

_Note!_: If your session gets disconnected you will need to reset these shell variables or use the IPs directly in the examples below.

Check that the SERVER1 and SERVER2 are set on both servers.
```
echo $SERVER{1,2}
```


**Installation of Java and FDT**

On Both servers:
```
# install java
sudo yum install -y java-1.8.0-openjdk.x86_64
java -version

# get FDT
wget https://github.com/fast-data-transfer/fdt/releases/download/0.25.1/fdt.jar
java -jar fdt.jar -version
```


**Help**

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

**Recursive copy**

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

For some fields of science datasets are described as a list of files with some additional metadata, hence this example.

The user can define a list of files (one filename per line) to be transferred. FDT will detect if the files are located on multiple devices and will use a dedicated thread for each device.

On SERVER2
```
java -jar fdt.jar -S
```

ON SERVER1
```
# create files and list first. User /dev/zero if /dev/urandom takes too long.
for i in {0..9}; do echo $i; dd if=/dev/urandom of=/tmp/f${i} count=204800; done
ls -1 /tmp/f* > file_list.txt
# do transfer from SERVER1 to SERVER2
java -jar fdt.jar -fl ./file_list.txt -c $SERVER2 -d /home/fdt/
```


**Test network connectivity**

To test the network connectivity one can start a transfer of data from /dev/zero on the server (SERVER1) to /dev/null on the client (SERVER2) using 10 streams in blocking mode, for both the server and the client with 8 MBytes buffers. The server will stop after the test is finished.

On the SERVER2 (server):
```
java -jar fdt.jar -bio -bs 8M -f $SERVER1 -S
```

On SERVER1 (client):
```
java -jar fdt.jar -c $SERVER2 -bio -P 10 -d /dev/null /dev/zero
```

_Nettest_

More recently we introduced a simpler way to do this with the `-nettest` flag.

```
# SERVER2
java -jar fdt.jar -f $SERVER1 -S
# SERVER1
java -jar fdt.jar -c $SERVER2 -nettest
```

_SCP mode_

On SERVER1:
```
java -jar fdt.jar -bio -P 10 /dev/zero fdt@$SERVER2:/dev/null
```


**Test local read/write performance**

DDCopy can be user to test the local read/write performance of local disks.

One can test the read speed with the following command. It copies the entire partition /dev/sda1 to /dev/null reporting by default every 2 seconds the I/O speed. The `sudo` is needed here because the fdt user does not have the permissions to read the device.

On SERVER1:
```
sudo java -cp fdt.jar lia.util.net.common.DDCopy if=/dev/sda1 of=/dev/null
```

To test the write speed of the file system we will use a 1GB file read from /dev/zero. The operating system will sync() the data to the disk. The data are read and written using 10MB buffers.

On SERVER1:
```
java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/fdt/1GBTestFile bs=10M count=100 flags=NOSYNC
```

OR

On SERVER2:
```
java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/fdt/1GBTestFile bs=1M bn=10 count=100 flags=NOSYNC
```

**Launch FDT as Agent**

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

**FDT Plugin**


FDT allows to load user defined classes for Pre- and Post-Processing of file transfers. This functionality can be used to easily interface FDT with storage systems and to implement additional Access Control List (ACL) to files transfered by FDT. It can also be used for packing, compression or customized integrity check.

The user can define its own syntax for managing files on different storage systems. The implementation for the Pre/Post Processing interfaces allows the user to define the mechanism to perform local staging or to move the transfered files to a storage system after they are transferred by FDT. The two procedures act as filters for the source and destination fields in the FDT syntax.

In the plugin shown below, the files will be first zipped in <filename>.zip by the PreZipFilter and their names will be changed in the ProcessorInfo and returned to the FDT client. Then <filename>.zip will be transferred to the destination where the PostZipFilter will uzip it and delete the zip file.

https://github.com/fast-data-transfer/fdt-plugins/tree/master/ZipFilter


On both servers:
```
# Install devel package
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
# Get Post and Pre ZipFilter
wget https://raw.githubusercontent.com/fast-data-transfer/fdt-plugins/master/ZipFilter/PreZipFilter.java
wget https://raw.githubusercontent.com/fast-data-transfer/fdt-plugins/master/ZipFilter/PostZipFilter.java
# Build and append to fdt.jar
javac -classpath fdt.jar *.java
```

On SERVER1:
```
mkdir zipTest && cd zipTest
for i in `seq 100`; do
  for j in `seq 1000`; do echo "local data"; done > $i.data
done
du -sb  ## REMEMBER THIS NUMBER!
cd ../
```

On SERVER2:
```
java -classpath fdt.jar:. lia.util.net.copy.FDT -postFilters PostZipFilter
```

On SERVER1:
```
java -classpath fdt.jar:. lia.util.net.copy.FDT -preFilters PreZipFilter -c $SERVER2 -d /tmp/ zipTest/*
```


