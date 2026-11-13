# ns8-monica

This is a template module for [NethServer 8](https://github.com/NethServer/ns8-core) for [Monica](https://github.com/monicahq/monica/).

## Install

Instantiate the module with:

```shell
add-module ghcr.io/geniusdynamics/monica:latest 1
```

The output of the command will return the instance name. Output example:

```json
{"module_id": "monica1", "image_name": "monica", "image_url": "ghcr.io/geniusdynamics/monica:latest"}
```

## Configure

Let's assume that the Monica instance is named `monica1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)

Example:

```shell
api-cli run configure-module --agent module/monica1 --data - <<EOF
{
  "host": "monica.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- start and configure the Monica instance
- configure a virtual host for Traefik to access the instance

## Get the configuration

You can retrieve the configuration with:

```shell
api-cli run get-configuration --agent module/monica1
```

## Uninstall

To uninstall the instance:

```shell
remove-module --no-preserve monica1
```

## Update

To update the instance:

```shell
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/monica:latest","instances":["monica1"],"force":true}'
```
    
## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the `configure-module` action input: they are discovered by looking at some Redis keys. To ensure the module is always up-to-date with the centralized [smarthost setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time Monica starts, the command `bin/discover-smarthost` runs and refreshes the `state/smarthost.env` file with fresh values from Redis.

Furthermore, if smarthost setup is changed when Monica is already running, the event handler `events/smarthost-changed/10reload_services` restarts the main module service.

See also the `systemd/user/monica.service` file.

This setting discovery is just an example to understand how the module is expected to work: it can be rewritten or discarded completely.

## Debug

Some CLI commands are needed to debug:

- The module runs under an agent that initiates a lot of environment variables (in `/home/monica1/.config/state`). It could be nice to verify them on the root terminal:

  ```shell
  runagent -m monica1 env
  ```

- You can become runagent for testing scripts and initiate all environment variables:

  ```shell
  runagent -m monica1
  ```

  The path becomes:

  ```shell
  echo $PATH
  /home/monica1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
  ```

- If you want to debug a container or see the environment inside:

  ```shell
  runagent -m monica1
  podman ps
  ```

  Example output:

  ```
  CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
  d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
  d8df02bf6f4a  docker.io/library/postgres:15.5-alpine3.19          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  postgresql-app
  9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  monica-app
  ```

- You can see what environment variables are inside the container:

  ```shell
  podman exec monica-app env
  ```

  Example output:

  ```
  TERM=xterm
  container=podman
  NGINX_VERSION=1.24.0
  PKG_RELEASE=1
  NJS_VERSION=0.7.12
  NGINX_IMAGE=docker.io/nginx:stable-alpine3.17
  CONFIG_DATABASE_URI="postgresql://postgres:Nethesis,1234@127.0.0.1:5432/toto"
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  HOME=/root
  ```

- You can run a shell inside the container:

  ```shell
  podman exec -ti monica-app sh
  / #
  ```
## Testing

Test the module using the `test-module.sh` script:

```shell
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/monica:latest
```

The tests are made using [Robot Framework](https://robotframework.org/).

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- Add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to the NS8 Weblate project
