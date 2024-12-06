# Oracle Database 23ai (23.5.0.0) Free Container Image Documentation

Oracle Database 23ai Free is the free edition of the industry-leading database. The Oracle Database 23ai Free Container Image contains Oracle Database 23ai (23.5.0.0) Free based on an Oracle Linux 8 base image.

Two flavors of the image are supported:

1.  The `Full` image: supports all the database features provided by Oracle Database 23ai Free.
2.  The `Lite` image: smaller image size with a stripped-down installation of the database.

The `Lite` image has a smaller storage footprint than the `Full` image (~80% image size reduction) and a substantial improvement in image pull time. This image is useful in CI/CD scenarios and for simpler use cases where advanced database features are not required.

For more information on Oracle Database 23ai Free, see: [https://oracle.com/database/free](https://oracle.com/database/free)

## Using the Full image

### Starting an Oracle Database 23ai Free container

The Oracle Database 23ai Free Container Image contains a pre-built database, so the **startup time is very fast**. Fast startup can be helpful in CI/CD scenarios. To start an Oracle Database Free container, run the following command where,`<oracle-db>` can be any custom name for the container:

```
docker run -d --name <oracle-db> container-registry.oracle.com/database/free:latest
```

When the container is started, a random password is generated for the `SYS, SYSTEM and PDBADMIN` users. This is termed the default password.

> [!NOTE]
> Throughout this document, words enclosed within angle brackets `<` `>` indicate variables in code lines.
> 
> To change the default password, see: "_Changing the Default Password for SYS User_"
> 
> To learn about advanced use cases, see: "_Custom Configurations_"
> 
> This document uses `docker` as the prescribed container runtime, but using contemporary commands is also anticipated to work.
> 
> The Oracle Enterprise Manager Database Express (OEM DB Express) is no longer supported with Oracle Database 23ai Free. Please use [SQL Developer](https://www.oracle.com/database/sqldeveloper/) instead, or [Oracle SQL Developer Extension for VSCode](https://marketplace.visualstudio.com/items?itemName=Oracle.sql-developer).

The Oracle Database is ready to use when the `STATUS` field shows `(healthy)` in the `docker ps` output.

### Custom Configurations

To facilitate custom configurations, the Oracle Database container provides configuration parameters that you can use when starting the container. For example, this is the detailed `docker run` command supporting all custom configurations:

```sh
docker run --name <container name> \
-P | -p <host port>:1521 \
-e ORACLE_PWD=<your database passwords> \
-e ORACLE_CHARACTERSET=<your character set> \
-e ENABLE_ARCHIVELOG=true \
-e ENABLE_FORCE_LOGGING=true \
-v [<host mount point>:]/opt/oracle/oradata \
container-registry.oracle.com/database/free:latest

Parameters:
   --name:        The name of the container (default: auto generated)
   -P | -p:       The port mapping of the host port to the container port.
                  Only one port is exposed: 1521 (Oracle Listener)
   -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDB_ADMIN password (default: auto generated)
   -e ORACLE_CHARACTERSET:
                  The character set to use when creating the database (default: AL32UTF8)
   -e ENABLE_ARCHIVELOG:
                  To enable archive log mode when creating the database (default: true)
   -e ENABLE_FORCE_LOGGING:
                  To enable force logging mode when creating the database (default: true)
   -v /opt/oracle/oradata
                  The data volume to use for the database.
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container.
                  If omitted the database will not be persisted over container recreation.
   -v /opt/oracle/scripts/startup
                  Optional: A volume with custom scripts to be run after database startup.
                  For further details see the "Running scripts after setup and on startup" section below.
   -v /opt/oracle/scripts/setup
                  Optional: A volume with custom scripts to be run after database setup.
                  For further details see the "Running scripts after setup and on startup" section below.
```

The supported configuration options are:

*   **ORACLE\_CHARACTERSET:** This parameter modifies the character set of the database. This parameter is optional, and the default value is set to AL32UTF8. Please note that, this parameter will set the character set only when a new database is created i.e. a host system directory is mounted using -v option while running the image (Please refer to _Mounting docker volume/host directory for database persistence_ section).

*   **ORACLE\_PWD:** This parameter modifies the password for the `SYS, SYSTEM and PDBADMIN` users. This parameter is optional, and the default value is randomly generated. Note: Using this option exposes the password as a container environment variable. Hence using docker secrets as described below is recommended.

*   **docker Secrets Support:** This option is supported only when **the docker runtime is being used** to run the Oracle Database 23ai Free container image. docker secrets is a secure mechanism of passing strings of text to the container, such as ssh-keys, or passwords. To specify the password for `SYS, SYSTEM and PDBADMIN` users securely, create a secret named `oracle_pwd` and a secret named `oracle_pwd_privkey` for the key to encrypt the password. The commands are as follows:

    *   Generate the private and public key files

        ```sh
          mkdir /opt/.secrets/
          cd /opt/.secrets
          openssl genrsa -out key.pem
          openssl rsa -in key.pem -out key.pub -pubout
        ```

    *   Create a text file with the unencrypted password Seed password and saving the /opt/.secrets/pwdfile.txt

        ```sh
          vi /opt/.secrets/pwdfile.txt
        ```

    *   Encrypt the password file using the public key generated above

        ```sh
          openssl pkeyutl -in /opt/.secrets/pwdfile.txt -out /opt/.secrets/pwdfile.enc -pubin -inkey /opt/.secrets/key.pub -encrypt
          rm -rf /opt/.secrets/pwdfile.txt
        ```

    *   Create docker secrets for the encrypted passwords and the private key generated above

        ```sh
          docker secret create oracle_pwd /opt/.secrets/pwdfile.enc
          docker secret create oracle_pwd_privkey /opt/.secrets/key.pem
        ```

    *   Pass the docker secrets to the container Pass the above created docker secrets `oracle_pwd` and `oracle_pwd_privkey` to the container with the `--secret` option

        ```sh
          docker run --name <container_name> --secret=oracle_pwd --secret=oracle_pwd_privkey container-registry.oracle.com/database/free:latest
        ```

*   **Mounting docker volume/host directory for database persistence:** To obtain database persistence, either a named docker volume or a host system directory can be mounted at the location `/opt/oracle/oradata` inside the container. The difference between these two options are as follows:

    1.  If a **docker volume** is mounted on the `/opt/oracle/oradata` location, then the volume is prepopulated with the data files already present in the image. In this case, the startup will be very fast, similar to starting the image without mount. These data files exist in the image to enable quick startup of the database.

        To use a docker volume for the data volume, run the following:

        ```sh
         docker run -d --name <oracle-db> \
          -v <OracleDBData>:/opt/oracle/oradata \
          container-registry.oracle.com/database/free:latest
        ```

    2.  If a **host system directory** is mounted on the `/opt/oracle/oradata` location, two things can happen :


    *   If the datafiles are present in the host system directory which is being mounted, then these files will overwrite the files present at `/opt/oracle/oradata` and the database startup will be very fast.

    *   If host system directory doesn't contain the datafiles the data files present at `/opt/oracle/oradata` will be overwritten , and **a new database setup will begin**. It takes a significant amount of time (approximately 10 minutes) to set up a fresh database.

        To use a directory on the host system for the data volume, run the following:

        ```sh
         docker run -d --name <oracle-db> \
          -v <writable_directory_path>:/opt/oracle/oradata \
          container-registry.oracle.com/database/free:latest
        ```


    **Note:** Cleaning up a mounted host directory can be done by changing into the directory location and executing `rm -rf * .*`

*   **ENABLE\_ARCHIVELOG and/or ENABLE\_FORCE\_LOGGING:** These options are enabled by default in the prebuilt database image. To disable these options, execute the relevant SQL commands using SQLPlus from within the database container.


### Changing the Default Password for SYS User

On the first startup of the container, a random password is generated for the database if a password is not provided by using the **\-e option**, as described in the "_Custom Configurations_" section.

To change the password for these accounts, use the `docker exec` command, to run the `setPassword.sh` script that is found in the container. Note that the container must be running before you run the script.

For example:

```sh
docker exec <oracle-db> ./setPassword.sh <your_password>
```

### Database Alert Logs

You can access the database alert log by using the following command, where `<oracle-db>` is the name of the container:

```sh
docker logs <oracle-db>
```

### Connecting to the Oracle Database Free Container

After the Oracle Database indicates that the container has started, and the `STATUS` field shows `(healthy),` client applications can connect to the database.

### Connecting from Within the Container

You can connect to the Oracle Database by running a SQL\*Plus command from within the container using one of the following commands:

```sh
docker exec -it <oracle-db> sqlplus sys/<your_password>@FREE as sysdba

docker exec -it <oracle-db> sqlplus system/<your_password>@FREE

docker exec -it <oracle-db> sqlplus pdbadmin/<your_password>@FREEPDB1
```

### Connecting from Outside the Container

By default, Oracle Database exposes port 1521 for Oracle client connections, using Oracle's SQL\*Net protocol. SQL\*Plus or any Oracle Database client software can be used to connect to the database from outside of the container.

To connect from outside of the container, start the container with the `-P` option, as described in the detailed docker run command in the "Custom Configurations" section.

Discover the mapped port by running the following command:

```sh
docker port <oracle-db>
```

To connect from outside of the container using SQL\*Plus, run the following commands:

```sh
# To connect to the database at the CDB$ROOT level as sysdba:
sqlplus sys/<your password>@//localhost:<port mapped to 1521>/FREE as sysdba
```

```sh
# To connect as non sysdba at the CDB$ROOT level:
sqlplus system/<your password>@//localhost:<port mapped to 1521>/FREE
```

```sh
# To connect to the default Pluggable Database (PDB) within the FREE Database:
sqlplus pdbadmin/<your password>@//localhost:<port mapped to 1521>/FREEPDB1
```

### Reusing the Existing Database

If the database is started using a host system directory mounted at an `/opt/oracle/oradata` location inside the container, as explained in the Custom Configuration section under "_Mounting docker volume/host directory for database persistence_", then the data files remain persistent even after the container is destroyed. Another container with the same data files can be started by reusing the host system directory.

To reuse this directory on the host system for the data volume, run the following commands:

```sh
docker run -d --name <oracle-db> -v \
                     <writable_host_directory_path>:/opt/oracle/oradata \
                     container-registry.oracle.com/database/free:latest
```

### Running Scripts After Setup and On Startup

The Container images can be configured to run scripts after setup and on startup. Currently, `.sh` and `.sql` extensions are supported. For post-setup scripts, mount the volume `/opt/oracle/scripts/setup` to include scripts in this directory. For post-startup scripts, mount the volume `/opt/oracle/scripts/startup` to include scripts in this directory.

After the database is set up or started, the scripts in those folders are run against the database in the container. SQL scripts are run as SYSDBA, and shell scripts are run as the current user. To ensure proper order for running scripts, Oracle recommends that you prefix your scripts with a number. For example: `01_users.sql`, `02_permissions.sql`, and so on.

**Note:** The setup scripts will not run in Free and Free Lite image as both comes with a pre-built database. If host mount point (empty directory) is provided, then a new database setup will start and setup scripts will run.

The following example mounts the local directory `/home/oracle/myScripts` to `/opt/oracle/scripts/startup`, which is then searched for custom startup scripts:

```sh
docker run -d --name <oracle-db> -v \
                     /home/oracle/myScripts:/opt/oracle/scripts/startup \
                     container-registry.oracle.com/database/free:latest
```

## Oracle True Cache

Oracle True Cache is an in-memory, consistent, and automatically managed cache for Oracle Database.

### Setting Up Network for Communication Between Primary Database and True Cache Container

*   Oracle Database Free True Cache container (True Cache container) and the Oracle Database Free Primary Database container (Primary Database container) must be on the same docker network to communicate with each other.
    Set up a docker network for inter-container communication using the following command which creates a bridge connection enabling communication between containers on the same host.

    ```sh
    docker network create tc_net
    ```

    Fetch the default subnet assigned to the preceding network by running the following command:

    ```sh
    docker inspect tc_net | grep -iw 'subnet'
    ```

    Pick any two IP addresses from the preceding subnet and assign one for the Primary Database container (say, PRI\_DB\_FREE\_IP) and the other for the True Cache container (say, TRU\_CC\_FREE\_IP).

    For communication across hosts, create a macvlan or ipvlan connection per [documentation](https://docs.docker.io/en/latest/markdown/docker-network-create.1.html).
    Specify the preceding docker network using the --net option to the docker run command of both the Primary Database container and the True Cache container as shown in following sections.


### Running Oracle Database Free True Cache in a Container

Ensure that the database password is specified as a docker secret to both the Primary and True Cache containers and not as an environment variable.

*   Launch the Oracle Database Free Primary Database container using the `docker run` command as follows:

    ```sh
    docker run -td --name pri-db-free \
    --hostname pri-db-free \
    --net=tc_net \
    --ip <PRI_DB_FREE_IP> \
    -p :1521 \
    --secret=oracle_pwd \
    --secret=oracle_pwd_privkey \
    --add-host="tru-cc-free:<TRU_CC_FREE_IP>" \
    -e ENABLE_ARCHIVELOG=true \
    -e ENABLE_FORCE_LOGGING=true \
    -v [<host mount point>:]/opt/oracle/oradata \
    container-registry.oracle.com/database/free:latest
    ```

    Ensure that your Primary Database container is up and running and in a healthy state.

*   Launch the Oracle Database Free True Cache container using the `docker run` command as follows:

    ```sh
    docker run -td --name tru-cc-free \
    --hostname tru-cc-free \
    --net=tc_net \
    --ip <TRU_CC_FREE_IP> \
    -p :1521 \
    --secret=oracle_pwd \
    --secret=oracle_pwd_privkey \
    --add-host="pri-db-free:<PRI_DB_FREE_IP>" \
    -e TRUE_CACHE=true \
    -e PRIMARY_DB_CONN_STR=<PRI_DB_FREE_IP>:1521/FREE \
    -e PDB_TC_SVCS="FREEPDB1:sales1:sales1_tc;FREEPDB1:sales2:sales2_tc;FREEPDB1:sales3:sales3_tc;FREEPDB1:sales4:sales4_tc" \
    -v [<host mount point>:]/opt/oracle/oradata \
    container-registry.oracle.com/database/free:latest
    ```

    **Note:** In above command, the list of Primary and True Cache services are mentioned using the string "FREEPDB1:sales1:sales1\_tc;FREEPDB1:sales2:sales2\_tc;FREEPDB1:sales3:sales3\_tc;FREEPDB1:sales4:sales4\_tc".
    The string consists of multiple entries in the format "::" and these entries are separated by ";".


## Using the Lite image

### Starting an Oracle Database 23ai Free Lite Container

Like the Full image, the Oracle Database 23ai Free Lite Container image also contains a pre-built database, so the **startup time is very fast**. Fast startup can be helpful in CI/CD scenarios. To start the container, run the following command where,`<oracle-db>` is the name of the container:

```sh
docker run -d --name <oracle-db> container-registry.oracle.com/database/free:23.5.0.0-lite
```

When the container is started, a random password is generated for the `SYS, SYSTEM, and PDBADMIN` users. This is termed as the default password.

The Oracle Database is ready to use when the `STATUS` field shows `(healthy)` in the `docker ps` output.

### Custom Configurations

To facilitate custom configurations, the Oracle Database container provides configuration parameters that you can use when starting the container. For example, this is the detailed `docker run` command supporting all custom configurations:

```sh
docker run --name <container name> \
-p <host port>:1521 -p  \
-e ORACLE_PDB=<your PDB name> \
-e ORACLE_PWD=<your database passwords> \
-v [<host mount point>:]/opt/oracle/oradata \
container-registry.oracle.com/database/free:23.5.0.0-lite

Parameters:
   --name:        The name of the container (default: auto generated).
   -p:            The port mapping of the host port to the container port.
                  The following ports is exposed: 1521 (Oracle Listener).

   -e ORACLE_PDB: The Oracle Database PDB name that should be used (default: FREEPDB1).
   -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDB_ADMIN password (default: auto generated).
   -v /opt/oracle/oradata
                  The data volume to use for the database.
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container.
                  If omitted the database will not be persisted over container recreation.
   -v /opt/oracle/scripts/startup
                  Optional: A volume with custom scripts to be run after database startup.
                  For further details see the "Running scripts after setup and on startup" section above.
```

**Note:** Oracle True Cache is not supported for the Lite image.

## Documentation Accessibility

For information about Oracle's commitment to accessibility, visit the Oracle Accessibility Program website at [https://www.oracle.com/corporate/accessibility/](https://www.oracle.com/corporate/accessibility/).
