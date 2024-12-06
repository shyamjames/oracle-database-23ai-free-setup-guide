# Setup Oracle Database 23ai Free with Docker for Development and Testing


# Running Oracle Database 23ai Free in a container

To run your Oracle Database 23ai Free container image use the `docker run` command as follows:

    docker run --name <container name> \
    -p <host port>:1521 \
    -e ORACLE_PWD=<your database passwords> \
    -e ORACLE_CHARACTERSET=<your character set> \
    -e ENABLE_ARCHIVELOG=true \
    -e ENABLE_FORCE_LOGGING=true \
    -v [<host mount point>:]/opt/oracle/oradata \
    oracle/database:23.5.0-free

    Parameters:
       --name:        The name of the container (default: auto generated)
       -p:            The port mapping of the host port to the container port.
                      Only one port is exposed: 1521 (Oracle Listener)
       -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDBADMIN password (default: auto generated)
       -e ORACLE_CHARACTERSET:
                      The character set to use when creating the database (default: AL32UTF8)
       -e ENABLE_ARCHIVELOG:
                      To enable archive log mode when creating the database (default: false)
       -e ENABLE_FORCE_LOGGING:
                      To enable force logging mode when creating the database (default: false)
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

Once the container has been started and the database created you can connect to it just like to any other database:

    sqlplus sys/<your password>@//localhost:1521/FREE as sysdba
    sqlplus system/<your password>@//localhost:1521/FREE
    sqlplus pdbadmin/<your password>@//localhost:1521/FREEPDB1

On the first startup of the container a random password will be generated for the database if not provided. The password for those accounts can be changed via the `docker exec` command. **Note**, the container has to be running:

    docker exec <container name> /opt/oracle/setPassword.sh <your password>

**Important Note:**
The ORACLE_SID for Oracle Database 23ai Free is always `FREE` and the PDB_NAME is always `FREEPDB1`. They cannot be changed, hence there
