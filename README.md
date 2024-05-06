
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

2.  Change to the repository directory:

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

## License

This project is licensed under the [MIT License](LICENSE).
