[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/monch1962/kube-pg-backup-restore)

# kube-pg-backup-restore
Backup/restore a Kube-hosted Postgres database to a file

## Option: Postgres running inside a container or Kubernetes cluster

This approach is ideal when your Postgres (or Postgres-compatible, such as CockroachDB) database is hosted inside containers. As of November 2020, this will often be the case when customers are using on-premise or hybrid clouds with sensitive data that can't be trusted to public cloud vendors.

### Backup

Using the following environment variables:
- `PG_POD` is the Kubernetes pod hosting the Postgres database, which needs to contain the `pg_dump` and `psql` executables
- `PG_USER` is a Postgres user with access to the database
- `PG_DB` is the Postgres database to be backed up
- `PG_BACKUP` is the file to hold the database backup (e.g. 'database.sql')

 `$ PG_POD=postgres PG_USER=user PG_DB=database kubectl exec $PG_POD -- bash -c "pg_dump -U $PG_USER PG_DB" > $PG_BACKUP`

## Restore

To restore the database using the same set of environment variables

`$ cat $PG_BACKUP | kubectl exec -i $PG_POD -- psql -U $PG_USER -d $PG_DB`

## Option: Postgres running on a remote system, or as a public cloud service (DBaaS)

In some instances, you either won't have access to the container running Postgres, or Postgres itself may be running as a service. This is a common scenario case for public cloud services e.g. AWS RDS, AWS Aurora, GCP CloudSQL, Azure Database for Postgres; you won't have access to a "database server" as such. 

In this case a different approach can be used bo backup/restore over TCP connections.

### Backup via TCP connection

You need a Postgres client installed somewhere with access to the Postgres instance. Running a Postgres client instance inside a Docker container such as `jbergknoff/postgresql-client` will often be the best approach e.g.:

`$ docker run -v /path/for/backup:/var/pgdata -it --rm --entrypoint pg_dump jbergknoff/postgresql-client -h $PG_HOST -U $PG_USER -f /var/pgdata/$PG_BACKUP $PG_DB`

This command will store a backup of the `DATABASE` database running on `HOST` to the file `/path/for/backup/mydump.sql`. The Postgres user `USER` will need to have the necessary database admin privileges to perform the backup.

If running inside Kubernetes, this same functionality could be implemented as a Kubernetes Batch job and/or potentially triggered via CI. Consult with your DevOps team to get this implemented

### Restore via TCP connection

`$ docker run -v /path/for/backup:/var/pgdata -it --rm --entrypoint psql jbergknoff/postgresql-client -h $HOST -U $PG_USER $PG_DB < /var/pgdata/$PG_BACKUP`

This command is the reverse of the backup option above. It will restore a backup located at `/path/for/backup/mydump.sql` to the `DATABASE` database running on `HOST`. The Postgres user `USER` will need to have the necessary database admin privileges to perform the restore.

If this command is run inside a Kubernetes environment, this functionality could be implemented as a Kubernetes Batch job and/or potentially triggered via CI. Consult with your DevOps team to get this implemented.
