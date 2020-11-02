[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/monch1962/kube-pg-backup-restore)

# kube-pg-backup-restore
Backup/restore a Kube-hosted Postgres database to a file

## Backup

Using the following environment variables:
- `PG_POD` is the Kubernetes pod hosting the Postgres database, which needs to contain the `pg_dump` and `psql` executables
- `PG_USER` is a Postgres user with access to the database
- `PG_DB` is the Postgres database to be backed up
- `PG_BACKUP` is the file to hold the database backup (e.g. 'database.sql')

 `$ PG_POD=postgres PG_USER=user PG_DB=database kubectl exec $PG_POD -- bash -c "pg_dump -U $PG_USER PG_DB" > $PG_BACKUP`

## Restore

To restore the database using the same set of environment variables

`$ cat $PG_BACKUP | kubectl exec -i $PG_POD -- psql -U $PG_USER -d $PG_DB`