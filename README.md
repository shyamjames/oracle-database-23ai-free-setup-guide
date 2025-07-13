# Running an Oracle Database in a Docker Container Using Docker Compose

This guide outlines how to set up and manage an Oracle Database in a Docker container using Docker Compose. This setup is ideal for development, testing, or learning purposes.

## Docker Compose Configuration

The `docker-compose.yml` file defines a service named `oracle-db` with the following key configurations:

### Service Configuration

- Image: `container-registry.oracle.com/database/free:latest`, this image provides a free version of the Oracle Database.
- Ports Exposed:
  - `1521`: Oracle Database listener port
  - `5500`: Oracle Enterprise Manager Express port
- Environment Variables:
  - `ORACLE_PWD`: Database password (e.g., `Welcome1234`)
  - `ORACLE_CHARACTERSET`: Character set for the database (e.g., `AL32UTF8`)

- Volume: Mounts the `./data` directory on the host to `/opt/oracle/oradata` in the container. This ensures data persistence.
- Network: Utilizes the `oracle-net` bridge network.
- Restart Policy: Restarts automatically unless explicitly stopped.

### Network Configuration

The `oracle-net` bridge network is defined in the `docker-compose.yml` file. This network allows the Oracle Database container to communicate with other containers on the same network.

## Running the Oracle Database Container

### Step 1: Prerequisites

Before running the Oracle Database container, ensure that you have Docker and Docker Compose installed on your system. You can download Docker Desktop from the official website: [Docker Desktop](https://www.docker.com/products/docker-desktop).

### Step 2: Clone the Repository

Clone this repository to your local machine using the following command:

```bash
git clone https://github.com/shyamjames/oracle-database-23ai-free-setup-guide.git
```

### Step 3: Set the Oracle Database Password

Edit the `docker-compose.yml` file and set the `ORACLE_PWD` environment variable to your desired password. For example:

```yaml
environment:
  - ORACLE_PWD=Welcome1234
```

### Step 4: Start the Oracle Database Container

To run the Oracle Database container, execute the following command in the same directory as the `docker-compose.yml` file:

```bash
docker-compose up -d
```

This command will download the necessary Docker image and start the Oracle Database container in detached mode.

### Step 5: Verify the Container Status

You can verify that the Oracle Database container is running by executing the following command:

```bash
docker-compose ps
```

The output should show the status of the `oracle-db` service as `Up`.

### Step 6: Connect to the Oracle Database

You can connect to the Oracle Database using a SQL client tool such as SQL\*Plus, SQLcl, or SQL Developer. Use the following connection details:

- Hostname: `localhost`
- Port: `1521`
- Service Name: `ORCLCDB.localdomain`
- Username: `sys as sysdba`
- Password: The password you set in the `docker-compose.yml` file

### Step 7: Stop the Oracle Database Container

To stop the Oracle Database container, execute the following command:

```bash
docker-compose down
```

This command will stop and remove the container while preserving the data in the `./data` directory on the host.

## Conclusion

By following this guide, you can easily set up and manage an Oracle Database in a Docker container using Docker Compose. This setup provides a convenient way to run an Oracle Database for development, testing, or learning purposes. The configuration can be customized for your specific development or testing needs.

### Further Resources

For more information on running Oracle Databases in Docker containers, refer to the official Oracle documentation: [Oracle Database 23ai (23.5.0.0) Free Container Image Documentation](https://container-registry.oracle.com/ords/f?p=113:4:13343733312256:::4:P4_REPOSITORY,AI_REPOSITORY,AI_REPOSITORY_NAME,P4_REPOSITORY_NAME,P4_EULA_ID,P4_BUSINESS_AREA_ID:1863,1863,Oracle%20Database%20Free,Oracle%20Database%20Free,1,0&cs=3pUu61oEtU0XJnPrPK0l7Wzy45alk-QUkECsiNkPe_bLGewdYwIYU9wdc6yHsbP4Qy2B-kn4TqZAAYLFPQIR1Yw).

## License

This repository is licensed under the MIT license. See `LICENSE` for more information.

## Contributing

Wanna contribute? Feel free to submit a pull request!
