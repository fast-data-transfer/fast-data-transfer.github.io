[[Home](index.md)]  [[Performance Tests](perf-disk-to-disk.md)]

[[Examples](doc-examples.md)]  [[Security](doc-security.md)]    [[User's Extensions](doc-user-extensions.md)]    [[System Tuning](doc-system-tuning.md)]

### FDT
**FDT** can be used in one of these five modes:
* **Server**: java -jar fdt.jar [ OPTIONS ]
* **Client**: java -jar fdt.jar [ OPTIONS ] -c <host> [file1 ...]|[-fl <fileList>] -d <destinationDirectory>
* **SCP**: java -jar fdt.jar [ OPTIONS ] [[[user@][host1:]]file1 [[[user@][host2:]]file2
* **Coordinator**: java -jar fdt.jar [OPTIONS] -c <host> -d <destinationDirectory> -sID <sessionID>
* **List Files**: java -jar fdt.jar [OPTIONS] -c <host> -ls <ls-path>
In Server mode the FDT will start listening for incoming client connections. The server may or may not stop after the last client finishes the transfer. In Client mode the client will connect to the specified host, where an FDT Server is expected to be running. The client can either read or write file from/to the server. 

In the SCP (Secure Copy) mode the local FDT instance will use SSH to start/stop the FDT server and/or client.  The security is based on ssh credentials. The server started in this mode will accept connections **ONLY** from the "SCP" client. It is possible to restrict the access for the FDT Servers started from the command line using the -f option. The option accepts a list of IP addresses separated by ':'. 

The OPTIONS currently supported may be server or client specific, or may be used in both modes.

Common  options used for both server and client:

**-gsi** enables GSI authentication scheme in FDT. When started in server mode the FDT will open two TCP ports: one for GSI authentication and the other one for data channels

**-gsip \<GSICtrlPort>** specifies the TCP port used for GSI authentication. Default value is 54320.

**-p \<portNo>** specifies the TCP port to be used (for the server it is the port used to listen on; for the client the port to connect to). The default port is 54321.

**-preFilters f1,...,fn** User defined preProcessing filters. The classes specified by the f1,...,fn paramenters must be in the classpath. The prePRocessing filters must be defined in the FDT "sender" command line. Please see the User's Filters section for more details and examples.

**-postFilters f1,...,fn** User defined postProcessing filters. The classes specified by the f1,...,fn paramenters must be in the classpath. The postPRocessing filters must be defined in the FDT "receiver" command line. Please see the User's Filters section for more details and examples.

**-bio** Blocking I/O mode. n this mode every channel (socket) will be configured to send/receive data synchronously and FDT will use one thread per channel. By default, non-blocking I/O will be used. On some platforms/systems the throughtput can be slightly higher in blocking I/O mode. The limitation in the blocking mode is the maximum number of threads that can be used and, for very high numbers of streams (thousands), the CPU used by the kernel for scheduling the threads. By default, FDT will use non-blocking mode.

**-iof \<iof>** Non-blocking I/O retry factor. In non-blocking mode every read/write operation which returns 0, will be repeated up to <iof> times before waiting for I/O readiness. By default this value is set to 1, which means that every network read/write operation will return in the select() (which can also be poll()/epoll()) if no more data can be processed by the underlying channel(socket). The default value should work fine on most of the systems, but values of 2 or 3, may increase the throughput on some systems. Values higher than 5 will only increase the CPU system usage, without any gain in performance. 

**-limit \<rate>** Restrict the transfer speed at the specified rate. K (KiloBytes/s), M (MegaBytes/s) or G (GigaBytes/s) may be used as suffixes. When this parameter is specified in the server it represents the maximum transfer rate for every FDT session. If the parameter is specified in both the server and the client, the minimum value between them will be used.

**-md5** Enables MD5 checksum for every file involved in the transfer. The flag may be specified for both client and server, but it will be used by the "sender" session only. When the transfer finishes the list will be sent to the "receiver" and it will be printed in a `md5sum`-like mode

**-printStats** Various statistics about buffer pools, sessions, etc will be printed

**-v** Verbose. Multiple 'v'-s (up to three) may be used to increment the verbosity level.Maximum level is three (-vvv) which corresponds to Level.FINEST for the standard Java logging system used by FDT.

**-u, -update** Update. If a newer version of fdt.jar is available on the update server it will update the local copy 

**Server options:**

**-S** disable the standalone mode; when specified the FDT Server will stop after the last client finishes. By default, the server will continue to listen for incoming clients. This option is automatically passed to the server started in "SCP" mode. 

**-bs \<buffSize>** Size for the I/O buffers. K (KiloBytes) or M (MegaBytes) may be used as suffixes. The default value is 512K. If the number of clients or sockets is expected to be very high is better to decrease this value. The memory used by this buffers is directly mapped in the operating system memory pages. The memory used by this buffers is limited by the JVM and can be increased passing -XX:MaxDirectMemorySize=<X>m (e.g -XX:MaxDirectMemorySize=256m) to the 'java' command

**-f \<allowedIPsList>** A list of IP addresses allowed to connect to the server. Multiple IPs may be specified, separated by ':'

**Client options:**

**-c \<host>** connect to the specified host. If this parameter is missing the FDT will become server 
**-gsissh** used in the Secure Copy Mode to specify GSI authentication instead of normal SSH authentication scheme. The remote sshd servere must support GSI authentication. 


**-d \<dstDir>** The destination directory used to copy files. 

**-fl \<fileList>** a list of files. Must have only one file per line. 

**-pull** Pull mode. The client will receive the data from the server. 

**-N** disable Nagle algorithm 

**-ss \<wsz>** Set the TCP SO_SND_BUFFER size. M and K may be used as suffixes for Kilo/Mega. 

**-P \<noOfStreams>** Number of paralel streams to use. Default is 4.
	
### DDCopy
**DDCopy** is very similar to Unix `dd` command and can be used to test the local disks or file system. It is bundled in the fdt.jar and has the following syntax:

java -cp fdt.jar lia.util.net.common.DDCopy [ OPTIONS ] **if=\<sourceFile> of=\<destinationFile>**

where OPTIONS may be:

**bs=\<BufferSize>**     size of the buffer used for read/write. K or M (for KiloBytes/MegaBytes) may be used as suffixes. Default is 4K

**bn=\<NoOfBuffers>**     Number of buffers used to readv()/writev() at once. If this parameter is 1, or is missing DDCopy will read()/write() a single buffer at a time, otherwise the readv()/writev() will be used. Default is 1

**count=\<count>**        Number of "blocks" to write. A "block" is represents how much data is read/write. The size of a "block" is: <BufferSize>*<BuffersNumber>. If <count> <= 0 the copy stops when EOF is reached reading the <SourceFile>. The default is 0

**statsdelay=\<seconds>**  Number of seconds between intermediate reports. Default is 2 seconds. If <seconds> <= 0 no intermediate reports will be printed

**flags=\<flag>**          The <flag> field can have of the following values:
                          **SYNC**    For every write both data and metadata are written synchronously
                          **DSYCN**  Same as SYNC, but only the data is written synchronously.
                          **NOSYNC** The sync() is left to be done by the underlying OS
                         The default value is **DSYNC**

**rformat=\<rformat>**     Report format. Possible values are:
                            K - KiloBytes
                            M - MegaBytes
                            G - GigaBytes
                            T - TeraBytes
                            P - PetaBytes
                         The default value is self adjusted. If the factor is too big only 0s will be displayed
