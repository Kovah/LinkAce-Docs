---
title: Application Backups
weight: 150
---

LinkAce provides an easy way to back up the whole application including the database to the local filesystem, or any S3-compatible service, including Amazon AWS S3, Minio or Backblaze B2. Under the hood LinkAce uses the [**Spatie Backup package**](https://spatie.be/docs/laravel-backup/v8/introduction). Please consult the documentation of the package for advanced configuration.

By default, the backup system will periodically purge old backups. For more details, [read the default configuration](https://spatie.be/docs/laravel-backup/v6/cleaning-up-old-backups/overview#determining-which-backups-should-be-deleted).


## Configure the backups

{{< alert type="warning" >}}
Please note that the [system cron]({{< relref path="docs/v2/configuration/system-settings.md#system-cron" >}}) should be configured for automatic backups to work correctly. However, you may also create backups manually, see below.
{{</ alert >}}

### Edit your .env file / environment variable

Add the following line to your .env file or environment variables:

```
BACKUP_ENABLED=true
```

### Edit your docker-compose.yml

First, create a backup folder to store the backups. The folder must be writable for other users, otherwise LinkAce won't be able to properly store the resulting files.

```bash
$ mkdir ./backups
$ chmod 0766 ./backups
```

Now, open the `docker-compose.yml` file and remove the `#` in front of the `- ./backups:/app/storage/app/backups` line like this:

```yaml {hl_lines=7}
services:
  # --- LinkAce
  app:
    image: docker.io/linkace/linkace:latest
    volumes:
      - ./.env:/app/.env
      - ./backups:/app/storage/app/backups
```

If you run a custom Docker-based setup (e.g. via Portainer), please adjust your container to mount a writable directory into `/app/storage/app/backups`.

That's it. LinkAce will now create backups every night at 2am.

---

## Configuration for LinkAce backups

The following settings allow you to configure backups for LinkAce. All settings go to your .env file:

| .env setting | Possible values | Default value | Description |
|:--|:--|:--|:--|
| `BACKUP_ENABLED` | `true`, `false` | `false` | Set to `true` to enable the application backups |
| `BACKUP_DISK` | `local_backups`, `s3` | `local_backups` | The storage for backups: `local_backups` saves the files to `/storage/app/backups`, `s3` saves to your configured S3 storage |
| `BACKUP_CLEAN_HOUR` | 24-hour time string | `01:00` | Set the hour for cleaning old backups and creating a new one. Format must be in 24-hour time like '03:00' for 3 am or '14:00' for 2 pm. |
| `BACKUP_RUN_HOUR` | 24-hour time string | `02:00` | See above |
| `BACKUP_NOTIFICATIONS_ENABLED` | `true`, `false` | `true` | Set this to `false` if you don't want to receive any email notifications for cleanups or backups. |
| `BACKUP_NOTIFICATION_EMAIL` | any email address | `your@email.com` | Set a valid email address to receive notification about backups. |
| `BACKUP_MAX_SIZE` | any number | `265` | The maximum size of all backups in Megabyte. Once reached the _oldest_ backups will be deleted. |
| `BACKUP_ARCHIVE_PASSWORD` | any string | none | Protect your backups with a password. |

The .env file or your environment variables should look like this: 

```bash
BACKUP_ENABLED=true
BACKUP_DISK=s3
AWS_ACCESS_KEY_ID=j85nRkzOgnlGc...
AWS_SECRET_ACCESS_KEY=DA5nHUT2B2B...
AWS_DEFAULT_REGION=eu-central-1
AWS_BUCKET=linkace
```

### Backing up to any s3-compatible storage

To back up LinkAce to S3, add the following settings to your .env file:

| .env setting | Possible values | Default value | Description |
|:--|:--|:--|:--|
| `AWS_URL` | A valid URL | `empty` | The URL to access the AWS service or storage bucket. Often used for serving content via CloudFront.  |
| `AWS_ENDPOINT` | A valid URL | `empty` | The custom endpoint URL for AWS services, used when working with non-standard AWS setups (e.g., local S3 emulators).  |
| `AWS_BUCKET` | any string | `empty` | The name of the S3 bucket where files will be stored or retrieved. |
| `AWS_ACCESS_KEY_ID` | any string | `empty` | Your AWS access key ID, used for authentication to AWS services. |
| `AWS_SECRET_ACCESS_KEY` | any string | `empty` | Your AWS secret access key, used alongside the access key ID for secure authentication. |
| `AWS_DEFAULT_REGION` | any string | `empty` | The AWS region where your bucket or resources are located (e.g., `us-east-1`). |
| `AWS_USE_PATH_STYLE_ENDPOINT` | `true`, `false` | `false` | Determines whether to use path-style (`true`) or virtual-hosted-style (`false`) URLs for S3 buckets.  |


The .env file or your environment variables should look like this:

```bash
BACKUP_ENABLED=true
BACKUP_DISK=s3
AWS_ACCESS_KEY_ID=j85nRkzOgnlGc...
AWS_SECRET_ACCESS_KEY=DA5nHUT2B2B...
AWS_DEFAULT_REGION=eu-central-1
AWS_BUCKET=linkace
```

### Using a third-party S3-compatible service

Instead of Amazon AWS S3, you can use any S3-compatible service. To connect to the service instead of AWS, you have to set the correct endpoint in your `.env` file. Near your other AWS_ settings, add the following line and replace the URL with your own service URL:

```bash
AWS_ENDPOINT=https://minio.example.com

# some non-AWS services use paths to reference buckets and endpoints.
# If this is the case, add the following line:
AWS_USE_PATH_STYLE_ENDPOINT=true
```

---

## Manually creating backups

Instead of relying on the automated backup process, you may run backups manually:

**Run a backup via Docker**

```
$ docker exec linkace-app-1 php artisan backup:run
```

**Run a backup via PHP**

```
$ cd /path/to/your/linkace
$ php artisan backup:run
```
