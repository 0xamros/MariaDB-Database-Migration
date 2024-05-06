
# MariaDB Database Migration And Hourly Database Backup

# MariaDB Database Migration

This repository contains an Ansible playbook for migrating three databases from an existing MariaDB instance to three separate Docker containers running MariaDB.

## Prerequisites

-   Ansible installed on the control machine
-   Access to the existing MariaDB instance (root user and password)
-   Docker installed on the target machine

## Usage

1.  Clone this repository:
```
git clone https://github.com/0xamros/MariaDB-Database-Migration.git
```

3.  Change to the repository directory:

```
cd MariaDB-Database-Migration
```

3.  Edit the `vars` section in the `migrate_dbs.yml` playbook to specify the names of the databases you want to migrate:

```
vars:
  dbs:
    - db1
    - db2
    - db3
```

4.  Update the `login_user` and `login_password` variables in the playbook with the appropriate credentials for your existing MariaDB instance.
5.  Run the playbook:

```
ansible-playbook migrate_dbs.yml
```

This will perform the following steps:

1.  Dump each database specified in the `dbs` list to an SQL file in the `/tmp` directory on the control machine.
2.  Create a new Docker container for each database, named `mariadb_<database_name>`.
3.  Mount the dumped SQL file to the container's `/docker-entrypoint-initdb.d` directory, which is used by the MariaDB container to initialize the database on startup.
4.  Map the container's port to a host port in the range 3301-3303

    (e.g., `mariadb_db1` will be accessible on port 3301,
    `mariadb_db2` on port 3302,
    and `mariadb_db3` on port 3303).

After the playbook completes, you should have three new Docker containers running MariaDB, each initialized with one of the migrated databases.

## Notes

-   The playbook assumes that the target machine has Docker installed and the Docker service is running.
-   The playbook allows an empty root password for the MariaDB containers for simplicity. In a production environment, you should set a secure root password.
-   The playbook mounts the SQL files to the containers' `/docker-entrypoint-initdb.d` directory, which means the data will be lost if the containers are stopped and removed. For persistent data storage, you should configure Docker volumes or bind mounts.

# MariaDB Database Migration

For scheduling hourly backups of three MariaDB databases running in Docker containers.

## Usage

Run the playbook:

```bash
ansible-playbook backup.yml
```

This will schedule a cron job on the target machine to perform the following tasks every hour:

1. For each of the databases `db1`, `db2`, and `db3`, execute a `mysqldump` command inside the corresponding Docker container (`mariadb_db1`, `mariadb_db2`, and `mariadb_db3`).
2. The `mysqldump` command will create a backup file for the database in the `/backup` directory on the target machine.
3. The backup file will be named with the current date and time in the format `YYYYMMDDHHMISS.<database_name>.sql`.

After the playbook completes, you should see the cron job scheduled on the target machine, and hourly backups should start running.

## Notes

- The playbook assumes that the target machine has Docker installed and the Docker service is running.
- The playbook assumes that the MariaDB containers are named `mariadb_db1`, `mariadb_db2`, and `mariadb_db3`. If your container names are different, you will need to modify the playbook accordingly.
- The playbook creates a `/backup` directory on the target machine to store the backup files. You may want to adjust the backup location or configure persistent storage for the backup files.
- The playbook assumes that the MariaDB containers have a root user with an empty password. In a production environment, you should set a secure root password and modify the playbook accordingly.

## License

This project is licensed under the [MIT License](LICENSE).
