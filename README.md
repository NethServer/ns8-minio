# ns8-minio

[MinIO](https://min.io/) module for NS 8.
This is a template module for [NethServer 8](https://github.com/NethServer/ns8-core).

## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/minio:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "minio1", "image_name": "minio", "image_url": "ghcr.io/nethserver/minio:latest"}

## Configure

Let's assume that the minio instance is named `minio1`.

Launch `configure-module`, by setting the following parameters:
- `host_server`: MinIO API server host name
- `host_console`: MinIO UI server host name
- `lets_encrypt`: enable or disable Let's Encrypt certificate
- `user`: MinIO admin user name, default to `minioadmin` (optional)
- `password`: MinIO admin user password, default to `minioadmin` (optional)

Example:

    api-cli run module/minio1/configure-module --data '{"host_server": "myminio.nethserver.org", "host_console": "console.myminio.nethserver.org", "lets_encrypt": true}'

The above command will:
- start and configure the minio instance: default root credentials `minioadmin`:`minioadmin`
- configure traefik to access MinIO with a valid Let's Encrypt certificate

Send a test HTTP request to the minio backend service:

    curl https://myminio.nethserver.org

### Use MinIO as NS8 backup storage

You can configure a NS8 machine to backup data to a MinIO bucket.

First, create a bucket. You can do from the UI or using the command line, eg:
```
curl  https://dl.min.io/client/mc/release/linux-amd64/mc -o /usr/local/bin/mc
chmod a+x /usr/local/bin/mc
mc alias set minio https://myminio.nethserver.org minioadmin minioadmin --api S3v4
mc mb minio/test1
```

Use the UI to create a generic S3 backup repository and schedule a backup for it.

### Use an external disk as MinIO storage

You can configure a MinIO instance to use a local attached USB/SCSI disk.
Given a disk named `scsi-disk1`, follow these steps:
```
# Create a mount point for your disk:
$ mkdir -p /mnt/data

# Change fstab to automatically mount the disk at boot
$ echo '/dev/disk/by-id/scsi-disk1 /mnt/data ext4 defaults,nofail,discard 0 0' >> /etc/fstab
$ systemctl daemon-reload

# Mount the disk
$ mount /mnt/data

# Make sure the disk is accessible from Minio instance (eg. minio1)
$ chown minio1:minio1 /mnt/data/
```

From the module UI, setup the `Storage path` under the advanded section and set it to `/mnt/data`.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve minio1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/minio:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
