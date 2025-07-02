## Topics

- [Topics](#topics)
  - [Install Prerequisites](#install-prerequisites)
  - [Setup Docker Swarm](#setup-docker-swarm)
  - [Setup Traefik](#setup-traefik)
  - [Setup Portainer](#setup-portainer)
  - [Setup MariaDB](#setup-mariadb)
  - [Setup swarm-cron](#setup-swarm-cron)
  - [Setup bench](#setup-bench)
  - [Configure the bench](#configure-the-bench)
  - [Create-site](#create-site)
- [After adding one site add second site](#after-adding-one-site-add-second-site)
  - [Then re-deploy](#then-re-deploy)
  - [Create-site](#create-site-1)
  - [Backup and Restore](#backup-and-restore)
    - [Backup](#backup)
    - [Restore](#restore)
    - [Upload the backup to GitHub (Free solution)](#upload-the-backup-to-github-free-solution)

### Install Prerequisites

These steps are required for production server. For dind/local setup copy `dind-devcontainer` to `.devcontainer` and reopen in devcontainer.

Use files from `compose` directory.

Setup assumes you are using Debian based Linux distribution.

Update packages

```shell
apt-get update && apt-get dist-upgrade -y
```

Setup unattended upgrades

```shell
dpkg-reconfigure --priority=medium unattended-upgrades
```

Add non-root sudo user

```shell
sudo adduser ubuntu
sudo usermod -aG sudo ubuntu
curl -fsSL https://get.docker.com | bash
sudo usermod -aG docker ubuntu
su - ubuntu
```

Clone this repo

```shell
git clone https://github.com/faizanops3/custom_containers.git
cd custom_containers
```

Note: traefik and portainer yamls specified in the further commands are in `compose` directory.

### Setup Docker Swarm

Initialize swarm

```shell
docker swarm init
docker swarm init --advertise-addr=X.X.X.X
```

Note: Make sure the advertise-address does not change if you wish to add multiple nodes to this manager.

Comprehensive guide available at [dockerswarm.rocks](https://dockerswarm.rocks)

### Setup Traefik

More on [Traefik](https://dockerswarm.rocks/traefik/)

Label the master node to install Traefik

```shell
docker node update --label-add traefik-public.traefik-public-certificates=true $(docker info -f '{{.Swarm.NodeID}}')
```

Set email and traefik domain

```shell
export EMAIL=faizanops@alazka.ai
export TRAEFIK_DOMAIN=erp-traefik.bloomi5.com
# or for dind
export TRAEFIK_DOMAIN=traefik.localhost
```

Set `HASHED_PASSWORD`

admin123

```shell
export HASHED_PASSWORD=$(openssl passwd -apr1)
Password: $ enter your password here
Verifying - Password: $ re enter your password here
```

Install Traefik in production

```shell
docker stack deploy -c compose/traefik-host.yml traefik
```

### Setup Portainer

More on [portainer](https://dockerswarm.rocks/portainer)

Label the master node to install portainer

```shell
docker node update --label-add portainer.portainer-data=true $(docker info -f '{{.Swarm.NodeID}}')
```

Install Portainer in production

```shell
# Set domain
export PORTAINER_DOMAIN=erp-portainer.bloomi5.com
# Install
docker stack deploy -c compose/portainer.yml portainer
```

### Setup MariaDB

```sh
docker stack deploy -c compose/mariadb.yml mariadb
```

### Setup swarm-cron

```sh
docker stack deploy -c compose/swarm-cron.yml swarm-cron
```

### Setup bench

```sh
docker stack deploy -c compose/erpnext-multi-my.yml sales
```

> Here instead of `SITES` use `SITE1`, `SITE2`, ... , for multiple sites.

### Configure the bench

```sh
docker stack deploy -c compose/configure-erpnext.yml configure-sales
```

> remove after complete

### Create-site

```sh
docker stack deploy -c compose/create-site.yml create-site
```

> remove after complete

---

## After adding one site add second site

### Then re-deploy

```sh
docker stack deploy -c compose/erpnext.yml sales
```

> Uncheck -->> `Re-pull image and redeploy`

### Create-site

```sh
docker stack deploy -c compose/create-site.yml create-site
```

> remove after complete

---

### Backup and Restore

#### Backup

> It is already happening at every 6 hours cron

- It generates two files,
- at this dir `/home/frappe/frappe-bench/sites/<site-name>/private/backups`

**Example**

```bash
20250701_120009-sales_bloomi5_com-database.sql.gz
20250701_180010-sales_bloomi5_com-site_config_backup.json
```

> `sales_bloomi5_com` is the site name

#### Restore

1. Rename `20250701_180010-sales_bloomi5_com-site_config_backup.json` to `site_config.json`
2. and upload to this location `/home/frappe/frappe-bench/sites/<site-name>`.
3. upload the `20250701_120009-sales_bloomi5_com-database.sql.gz` to `/home/frappe/frappe-bench/sites/<site-name>/private/backups` as it is.
4. then run this command. `bench --site <site-name> restore /home/frappe/frappe-bench/sites/<site-name>/private/backups/<the-selected-file-name>.sql.gz`

#### Upload the backup to GitHub (Free solution)
