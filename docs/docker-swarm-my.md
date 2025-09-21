## Topics

### Install Prerequisites

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
git clone https://github.com/bloomi5/custom_containers.git
cd custom_containers
```

Note: traefik and portainer yamls specified in the further commands are in `compose` directory.

### Setup Docker Swarm

Initialize swarm

```shell
docker swarm init
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

- Doploy this with portainer.
- Dont add any env.
- Add Stack -> mariadb -> copy paste the compose/mariadb.yml -> Deploy.

```sh
docker stack deploy -c compose/mariadb.yml mariadb
```

### Setup swarm-cron

- Doploy this with portainer.
- Dont add any env.
- Add Stack -> swarm-cron -> copy paste the compose/swarm-cron.yml -> Deploy.

```sh
docker stack deploy -c compose/swarm-cron.yml swarm-cron
```

### Setup bench

- Doploy this with portainer.
- Add env.
- Add Stack -> sales -> copy paste the compose/erpnext-multi-bloomi5.yml -> Deploy.

> Here instead of `SITES` use `SITE1`, `SITE2`, ... , for multiple sites.

```sh
VERSION=v18
BENCH_NAME=sales
SITE1=`erpnext.bloomi5.com`
SITE2=`nomnomtails.bloomi5.com`
```

```sh
docker stack deploy -c compose/erpnext-multi-bloomi5.yml sales
```

> Wait for the containers to run, confirm in portainer.

### Configure the bench

- Doploy this with portainer.
- Add env.
- Add Stack -> configure -> copy paste the compose/configure-erpnext.yml -> Deploy.

> remove after complete.

```sh
VERSION=v18
BENCH_NAME=sales
```

```sh
docker stack deploy -c compose/configure-erpnext.yml configure
```

### Create-site

- Doploy this with portainer.
- Add env., and command **< your site name >**.
- if you have multiple site you have to run it multiple time per site.
- Add Stack -> create-site -> copy paste the compose/create-site.yml -> Deploy.

> remove after complete.

```sh
VERSION=v18
BENCH_NAME=sales
```

```sh
docker stack deploy -c compose/create-site.yml create-site
```

> remove after complete

### Backup and Restore

> Do not set cronjob it messes with swarm-cron.

#### Automated Backup and Restore

##### Backup

- Clone this repo in the vm: https://github.com/bloomi5/frappe-backup-repo
- Backup is done from GitHub Actions pipeline
- read the [README](https://github.com/bloomi5/frappe-backup-repo/blob/dev/README.md)

##### Restore

```sh
ubuntu@erp:~/frappe-backup-repo$ ./backup_restore_frappe.sh restore
Enter site name (e.g. sales.bloomi5.com): erpnext.bloomi5.com
Enter JSON filename (e.g. site_config_backup.json): 20250920_180001-erpnext_bloomi5_com-site_config_backup.json
Enter SQL filename (e.g. database.sql.gz): 20250920_180001-erpnext_bloomi5_com-database.sql.gz
Successfully copied 2.05kB to ab23023d8b33:/home/frappe/frappe-bench/sites/erpnext.bloomi5.com/site_config.json
Uploaded site_config.json to erpnext.bloomi5.com
Successfully copied 1.91MB to ab23023d8b33:/home/frappe/frappe-bench/sites/erpnext.bloomi5.com/private/backups/20250920_180001-erpnext_bloomi5_com-database.sql.gz
Uploaded 20250920_180001-erpnext_bloomi5_com-database.sql.gz to erpnext.bloomi5.com
Running bench restore inside container...
MySQL root password: <same password - maria db root>
App frappe already installed
erpnext.bloomi5.com: SystemSettings.enable_scheduler is UNSET
*** Scheduler is disabled ***
Site erpnext.bloomi5.com has been restored
Restore completed for erpnext.bloomi5.com using 20250920_180001-erpnext_bloomi5_com-database.sql.gz

```

#### Manuall

##### Backup

> It is already happening at every 6 hours cron

- It generates two files,
- at this dir `/home/frappe/frappe-bench/sites/<site-name>/private/backups`

**Example**

```bash
20250701_120009-sales_bloomi5_com-database.sql.gz
20250701_180010-sales_bloomi5_com-site_config_backup.json
```

> `sales_bloomi5_com` is the site name

##### Restore

1. Rename `20250701_180010-sales_bloomi5_com-site_config_backup.json` to `site_config.json`
2. and upload to this location `/home/frappe/frappe-bench/sites/<site-name>`.
3. upload the `20250701_120009-sales_bloomi5_com-database.sql.gz` to `/home/frappe/frappe-bench/sites/<site-name>/private/backups` as it is.
4. then run this command. `bench --site <site-name> restore /home/frappe/frappe-bench/sites/<site-name>/private/backups/<the-selected-file-name>.sql.gz`
