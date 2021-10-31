# MySQL backup role

This role adds a cron to backup MySQL & friends using mysqldump.

Optionally encrypts backups using provided GPG keys.

Optionally copies backups to object storage (S3 or GCS).

## Requirements

- `mysqldump` must be present on the system.
- [influxevent](https://github.com/devops-works/ansible-influxevent) must be
  installed if metrology is used 

## Role Variables

- `backup_mysql_exclude` (default: ""): `grep` exclude pattern for databases
- `backup_mysql_keep` (default: 30): how many backups do we keep locally
- `backup_mysql_s3bucket` (default: ""): S3 bucket to export backups to
- `backup_mysql_gcloudbucket` (default: ""): GCS bucket to export backups to
- `backup_mysql_destination` (default: "/var/backups/mysql/"): path to store
  local backups in
- `backup_mysql_cron_time` (default: "15 */2 * * 0-7"): cron entry for
  backups
- `backup_mysql_gpg_keys` (default: []): list of gpg keys URLS to
  encrypt backups for
- `backup_mysql_user`: backup user
- `backup_mysql_password`: backup password

## Encryption

If `backup_mysql_gpg_keys` is populated with GPG keys URLs, backups will
be encrypted for those keys. When no keys are provided, backup will not be
encrypted.

If encryption is used, **only encrypted backups will be pushed to object
storage**.

If encryption is not used, **unencrypted backups will be pushed to object
storage**.

Each object in the `backup_mysql_gpg_keys` list must have an `id` and a `url`
attribute:

```
mariadb_backup_gpg_keys:
  - url: https://gitlab.com/alice.gpg
    id: AAAA40FC29B8AAAA
  - url: https://gitlab.com/bob.gpg
    id: BBBBE221A5D7BBBB
```

## Object storage

Buckets must be set in `backup_mysql_s3bucket` or `backup_mysql_gcloudbucket`
if you want to offload backups to object storage.

This role expects objects storage to be already setup if you intend to use it.
`s3cmd` and `gsutil` are used respectively for AWS S3 and GCS.

See [ansible-gcloud](https://github.com/devops-works/ansible-gcloud) for GCS.

For GCS, we recomment configuring the bucket with:

- a lifecycle policy to expunge old backups (see Lifecycle in bucket settings)
- a retention policy to ensure backup immutability (see Protction in bucket
  settings)

## Metrology support

If `backup_mysql_influxevent` and `telegraf_output_influxdb` are set, the role
will use [`influxevent`](https://gitlab.com/devopsworks/tools/influxevent) to
send backup events to an influxdb server.

InfluxEvent must be installed if you intend to use it (see
https://github.com/devops-works/ansible-influxevent).

The following variables are required:

- `telegraf_output_influxdb.database`
- `telegraf_output_influxdb.urls`
- `telegraf_output_influxdb.username`
- `telegraf_output_influxdb.password`

See https://github.com/devops-works/ansible-telegraf#variables for more
information on those vars.

## Dependencies

None

## Warning

Try this role many times and ensure it fits your needs before using it for
production...

## License

MIT

## Author Information

[@devops-works](https://github.com/devops-works)

Patches welcome!
