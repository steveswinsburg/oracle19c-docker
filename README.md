# oracle19c-docker
A lightweight and configurable Oracle 19c docker image.

Oracle has introduced the concept of container databases (CDB) and pluggable databases (PDB). Containers are used for multi-tenancy and contain pluggable databases. Pluggable databases are what you are probably used to, a self contained database that you connect to. By default, this image creates one CDB, and one PDB within that CDB.

Note that Oracle is shifting away from an SID and using service names instead. PDB use service names, CDB use SIDs. However, usernames for CDB must start with C##, ie C##STEVE. So to make things simple, we are focusing on just using the single PDB.

I used to maintain a standalone repository for building this image. It was originally based on the official images, like my Oracle 12c one here: https://github.com/steveswinsburg/oracle12c-docker

Oracle have since improved their docker images and I have worked with Oracle on making the the memory configurable (see https://github.com/oracle/docker-images/issues/1575), so all we need now are simplified instructions.

Before you begin
----------------

1. Clone `https://github.com/oracle/docker-images`.
1. Download the Oracle Database 19c binary from http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html
    1. For X86, get `LINUX.X64_193000_db_home.zip`.
    1. For ARM, get `LINUX.ARM64_1919000_db_home.zip`.
1. Put the zip in the `OracleDatabase/SingleInstance/dockerfiles/19.3.0` directory. **Do not unzip it.**
1. In your docker application (e.g. Colima, Docker Desktop etc), ensure you have a large enough amount of memory allocated to docker. These instructions will set the total memory to 4000MB, so make sure Docker has a value higher than that.

Start your container environment
--------------------------------
These instructions assume you are using Colima. Start up colima with the `-e` flag and an editor will open where you can review your settings. Also note that these instructions are assuming v0.5.3 and MacOS >= 13 (Ventura).

`colima start -e`

**For x86:**
```
# Number of CPUs to be allocated to the virtual machine.
# Default: 2
cpu: 4
 
# Size of the disk in GiB to be allocated to the virtual machine.
# NOTE: changing this has no effect after the virtual machine has been created.
# Default: 60
disk: 100
 
# Size of the memory in GiB to be allocated to the virtual machine.
# Default: 2
memory: 8
 
# Architecture of the virtual machine (x86_64, aarch64, host).
# Default: host
arch: x86_64
 
# Container runtime to be used (docker, containerd).
# Default: docker
runtime: docker

...
 
# Virtual Machine type (qemu, vz)
# NOTE: this is macOS 13 only. For Linux and macOS <13.0, qemu is always used.
#
# vz is macOS virtualization framework and requires macOS 13
#
# Default: qemu
vmType: vz
 
# Volume mount driver for the virtual machine (virtiofs, 9p, sshfs).
#
# virtiofs is limited to macOS and vmType `vz`. It is the fastest of the options.
#
# 9p is the recommended and the most stable option for vmType `qemu`.
#
# sshfs is faster than 9p but the least reliable of the options (when there are lots
# of concurrent reads or writes).
#
# Default: virtiofs (for vz), sshfs (for qemu)
mountType: virtiofs
```

**For ARM:** 
```
# Number of CPUs to be allocated to the virtual machine.
# Default: 2
cpu: 4
 
# Size of the disk in GiB to be allocated to the virtual machine.
# NOTE: changing this has no effect after the virtual machine has been created.
# Default: 60
disk: 100
 
# Size of the memory in GiB to be allocated to the virtual machine.
# Default: 2
memory: 8
 
# Architecture of the virtual machine (x86_64, aarch64, host).
# Default: host
arch: aarch64
 
# Container runtime to be used (docker, containerd).
# Default: docker
runtime: docker

...
 
# Virtual Machine type (qemu, vz)
# NOTE: this is macOS 13 only. For Linux and macOS <13.0, qemu is always used.
#
# vz is macOS virtualization framework and requires macOS 13
#
# Default: qemu
vmType: vz

# Utilise rosetta for amd64 emulation (requires m1 mac and vmType `vz`)
# Default: false
rosetta: true
 
# Volume mount driver for the virtual machine (virtiofs, 9p, sshfs).
#
# virtiofs is limited to macOS and vmType `vz`. It is the fastest of the options.
#
# 9p is the recommended and the most stable option for vmType `qemu`.
#
# sshfs is faster than 9p but the least reliable of the options (when there are lots
# of concurrent reads or writes).
#
# Default: virtiofs (for vz), sshfs (for qemu)
mountType: virtiofs
```

Building
--------

````
cd OracleDatabase/SingleInstance/dockerfiles
./buildContainerImage.sh -v 19.3.0 -e
````

If the build fails saying you are out of space, check how much space you have available on your disk. If it looks ok, prune old Docker images via: 
`yes | docker image prune > /dev/null`

Running
-------

To use the sensible defaults:

```
docker run \
--name oracle19c \
-p 1521:1521 \
-p 5500:5500 \
-e ORACLE_PDB=orcl \
-e ORACLE_PWD=password \
-e INIT_SGA_SIZE=3000 \
-e INIT_PGA_SIZE=1000 \
-v /opt/oracle/oradata \
-d \
oracle/database:19.3.0-ee
```

On first run, the database will be created and setup for you. This will take about 10-15 minutes. Open the container logs and watch the progress. Then you can connect.

Optionally, you can use the following run command to avoid getting "No disk space" issues as you gradually insert more and more data into your database.

```
docker run \
--name oracle19c \
-p 1521:1521 -p 5500:5500 \
-e ORACLE_PDB=orcl \
-e ORACLE_PWD=password \
-e INIT_SGA_SIZE=3000 \
-e INIT_PGA_SIZE=1000 \
-v /Users/<your-username>/path/to/store/db/files/:/opt/oracle/oradata \
-d \
oracle/database:19.3.0-ee
```

Configuration
-------------

```
   --name:        The name of the container (default: auto generated)
   -p:            The port mapping of the host port to the container port.
                  Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express)
   -e ORACLE_SID: The Oracle Database SID that should be used (default: ORCLCDB)
   -e ORACLE_PDB: The Oracle Database Service Name that should be used (default: ORCLPDB1)
   -e ORACLE_PWD: The Oracle Database SYS password (default: auto generated)
   -e INIT_SGA_SIZE: The amount of SGA to allocate to Oracle. This should be 75% of the total memory you want Oracle to use. 
   -e INIT_PGA_SIZE: The amount of PGA to alloxate to oracle. This should be the remaining 25% of the total memory you want Oracle to use.  
   -e ORACLE_CHARACTERSET: The character set to use when creating the database (default: AL32UTF8)
   -v /opt/oracle/oradata
                  The data volume to use for the database.
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container!
                  If omitted the database will not be persisted over container recreation.
   -v /Users/<your-username>/path/to/store/db/files/:/opt/oracle/oradata
                  Mount the data volume into one of your local folders.
                  If omitted you might run into a "No disk space" issue at some point as your database keeps growing and docker does not resize its volume.
   -d:            Run in detached mode. You want this otherwise `Ctrl-C` will kill the container.
```

Note that if you do not specify INIT_SGA_SIZE and INIT_PGA_SIZE then Oracle will determine the memory to allocate based on the number of CPUs, available memory in your machine etc, and for desktop environments this could be from about 2000MB to 5000MB. If you want control over this, set the values.

Connecting to Oracle
--------------------

Once the container has been started you can connect to it like any other database. Note we are using `Service Name` and not the SID (since PDB uses Service Name).

For example, SQL Developer:
```
Hostname: localhost
Port: 1521
Service Name: <your service name>
Username: sys
Password: <your password>
Role: AS SYSDBA

```

Changing the password
---------------------

Note: If you did not set the `ORACLE_PWD` parameter, check the docker run output for the password.

The password for the SYS account can be changed via the `docker exec` command. Note, the container has to be running:

First run `docker ps` to get the container ID. Then run:
`docker exec <container id> ./setPassword.sh <new password>`

Getting a shell on the container
--------------------------------
First run `docker ps` to get the container ID. Then run:
`docker exec -it <container id> /bin/bash`

Or as root:
`docker exec -u 0 -it <container id> /bin/bash`

Using SQLPlus within the container
----------------------------------

Once on the container (see above), connect to SQLPlus like so:
```
sqlplus /nolog
connect sys/password@orcl as sysdba
```
