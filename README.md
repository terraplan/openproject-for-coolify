# OpenProject installation with Coolify

This repository contains the installation method for OpenProject using Coolify.


## Quick start (LOCALHOST)

First, you must clone the [openproject-docker-compose](https://github.com/christian-quisbert/openproject-for-coolify.git) repository:

```shell
git clone https://github.com/christian-quisbert/openproject-for-coolify.git
```
Copy the example `docker-compose.override.yml` file.\
Copy the example `.env` file and edit any values you want to change:

```shell
cp docker-compose.override.yml.example docker-compose.override.yml
cp .env.example .env
nano .env
```

If you are using the default value of OPDATA that is used in the ```.env.example``` you need to make sure that the folder exist, and you have the right permissions:

```shell
sudo mkdir -p /var/openproject/assets
sudo chown 1000:1000 -R /var/openproject/assets
```

Next you start up the containers in the background while making sure to pull the latest versions of all used images.

```shell
docker compose up -d --build --pull always
```
After a while, OpenProject should be up and running on `http://localhost:8080`. The default username and password is login: `admin`, and password: `admin`.
The `OPENPROJECT_HTTPS=false` environment variable explicitly disables HTTPS mode for the first startup. Without this, OpenProject assumes it's running behind HTTPS in production by default.


You will also need to provide a secure key for the `SECRET_KEY_BASE`, it should be unique and treated like a password.
This value is used to derive keys for the Rails application. Changing this value will invalidate all cookies, including sessions and 2FA remembering tokens.

### Customization

The `docker-compose.yml` file present in the repository can be adjusted to your convenience. But note that with each pull, it will be overwritten.
Best practice is to use the file `docker-compose.override.yml` for that case.
For instance you could mount specific configuration files, override environment variables, or switch off services you don't need.

Please refer to the official [Docker Compose documentation](https://docs.docker.com/compose/extends/) for more details.


### Collaboration server

The collaboration server is enabled by default when setting up this application.

> Important! Make sure to override the default secret by adjusting the docker-compose file or setting the `COLLABORATIVE_SERVER_SECRET` variable.


### HTTPS/SSL

On **LOCALHOST**, by default OpenProject starts with the HTTP option **disabled**.



### PORT

By default the port is bound to `0.0.0.0` means access to OpenProject will be public.
See below how to change that.

## Image configuration

OpenProject publishes `slim` containers that you should be using for this compose setup.
Please see https://www.openproject.org/docs/installation-and-operations/installation/docker/#available-containers for more information on the containers and versions we push.

## Environment variables Configuration

Environment variables can be added to `docker-compose.yml` under `x-op-app -> environment` to change
OpenProject's configuration. Some are already defined and can be changed via the environment.

You can pass those variables directly when starting the stack as follows.

```
VARIABLE=value docker-compose up -d
```

You can also put those variables into an `.env` file in your current working
directory, and Docker Compose will pick it up automatically. See `.env.example`
for details.


## APP_PORT

If you want to specify a different port, you can do so with:

```
APP_PORT=8080
```

If you don't want OpenProject to bind to `0.0.0.0` you can bind it to localhost only like this:

```
PORT=127.0.0.1:8080
```

## TAG

If you want to specify a custom tag for the OpenProject docker image, you can do so with:

```
TAG=my-docker-tag
```

## BIM edition

In order to install or change to BIM inside a Docker environment, please navigate to the [Docker Installation for OpenProject BIM](https://www.openproject.org/docs/installation-and-operations/bim-edition/#docker-installation-openproject-bim) paragraph at the BIM edition documentation.

## Upgrade

Retrieve any changes from the `openproject-docker-compose` repository:

    git pull origin stable/17

Build the control plane:

    docker-compose -f docker-compose.yml -f docker-compose.control.yml build

Take a backup of your existing postgresql data and openproject assets:

    docker-compose -f docker-compose.yml -f docker-compose.control.yml run backup

Run the upgrade:

    docker-compose -f docker-compose.yml -f docker-compose.control.yml run upgrade

Relaunch the containers, ensure you are pulling to use the latest version of the Docker images:

    docker compose up -d --build --pull always



## Backup

Switch off your current installation:

    docker-compose down

Build the control scripts:

    docker-compose -f docker-compose.yml -f docker-compose.control.yml build

Take a backup of your existing PostgreSQL data and OpenProject assets:

    docker-compose -f docker-compose.yml -f docker-compose.control.yml run backup

Restart your OpenProject installation

    docker-compose up -d



## Uninstall

If you want to stop the containers without removing them directly:

```bash
docker-compose stop
```

You can remove the container stack with:

```bash
docker-compose down
```

> [!NOTE]
> This will not remove your data which is persisted in named volumes, likely called `compose_opdata` (for attachments) and `compose_pgdata` (for the database).
> The exact name depends on the name of the directory where your `docker-compose.yml` and/or you `docker-compose.override.yml` files are stored (`compose` in this case).

If you want to start from scratch and remove the existing data you will have to remove these volumes via
`docker volume rm compose_opdata compose_pgdata`.

## Troubleshooting

You can look at the logs with:

    docker-compose logs -n 1000

For the complete documentation, please refer to https://docs.openproject.org/installation-and-operations/.

### Network issues

If you're running into weird network issues and timeouts such as the one described in
[OP#42802](https://community.openproject.org/work_packages/42802), you might have success in remove the two separate
frontend and backend networks. This might be connected to using podman for orchestration, although we haven't been able
to confirm this.

### SMTP setup fails: Network is unreachable.

Make sure your container has DNS resolution to access external SMTP server when set up as described in
[OP#44515](https://community.openproject.org/work_packages/44515).

```yml
worker:
  dns:
    - "Your DNS IP" # OR add a public DNS resolver like 8.8.8.8
```
