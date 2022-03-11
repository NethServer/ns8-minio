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

Example:

    api-cli run module/minio1/configure-module --data '{"host_server": "myminio.nethserver.org", "host_console": "console.myminio.nethserver.org", "lets_encrypt": true}'

The above command will:
- start and configure the minio instance: default root credentials `minioadmin`:`minioadmin`
- configure traefik to access MinIO with a valid Let's Encrypt certificate

Send a test HTTP request to the minio backend service:

    curl https://myminio.nethserver.org

## Uninstall

To uninstall the instance:

    remove-module --no-preserve minio1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/minio:latest

The tests are made using [Robot Framework](https://robotframework.org/)
