# oracle19c-docker
A lightweight and configurable Oracle 19c docker image.

Oracle has introduced the concept of container databases (CDB) and pluggable databases (PDB). Containers are used for multi-tenancy and contain pluggable databases. Pluggable databases are what you are probably used to, a self contained database that you connect to. By default, this image creates one CDB, and one PDB within that CDB.

Note that Oracle is shifting away from an SID and using service names instead. PDB use service names, CDB use SIDs. However, usernames for CDB must start with C##, ie C##STEVE. So to make things simple, we are focusing on just using the single PDB.

I used to maintain a standalone repository for building this image. It was originally based on the official images, like my Oracle 12c one here: https://github.com/steveswinsburg/oracle12c-docker

Oracle have since improved their docker images and I have sent a PR for making the memory configurable (https://github.com/oracle/docker-images/pull/1576), so all we need now are simplified instructions.

Before you begin
----------------

1. Clone `https://github.com/oracle/docker-images`.
1. Download the Oracle Database 19c binary `LINUX.X64_193000_db_home.zip` from http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html
1. Put the zip in the `OracleDatabase/SingleInstance/dockerfiles/19.3.0` directory. **Do not unzip it.**
1. If https://github.com/oracle/docker-images/pull/1576 is not yet merged, edit `OracleDatabase/SingleInstance/dockerfiles/19.3.0/dbca.rsp.tmpl`, and change `totalMemory=2048` to `totalMemory=4000` or whatever value you want.
1. In Docker Desktop, update the allocated memory to a value more than the value above.

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
-e ORACLE_MEM=4000 \
-v /opt/oracle/oradata \
-d \
oracle/database:19.3.0-ee
```

On first run, the database will be created and setup for you. This will take about 10-15 minutes. Open Docker Dashboard and watch the progress. Then you can connect.

Optionally, you can use the following run commmand to avoid getting "No disk space" issues as you gradually insert more and more data into your database.

```
docker run \
--name oracle19c \
-p 1521:1521 -p 5500:5500 \
-e ORACLE_PDB=orcl \
-e ORACLE_PWD=password \
-e ORACLE_MEM=4000 \
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
   -e ORACLE_MEM: The amount of memory in MB to allocate to Oracle. If you bump this up too much you might need to change your Docker settings to allocate more memory to Docker (default: 2048)
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
