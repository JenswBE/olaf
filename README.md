# Olaf the Great NAS
Config for my home NAS

## Setup
### Install CoreOS
1. Download the latest version of the [config transpiler](https://github.com/coreos/container-linux-config-transpiler/)
2. Complete config `olaf-clc.yml`
3. Transpile the config using `ct -strict < olaf-clc.yml > olaf-clc.json`
4. Install CoreOS using `coreos-install -d /dev/sdX -i olaf-clc.json`
5. Reboot
6. Change hostname with `sudo hostnamectl set-hostname olaf`
7. Create opt folders with `sudo mkdir -p /opt/bin /opt/config`

### Docker Compose
Use [following instructions](https://docs.docker.com/compose/install/#install-compose) and install Docker compose at `/opt/bin/docker-compose`

### Basic setup
1. Clone this repo with `git clone https://github.com/JensWillemsens/olaf.git`
2. Copy file `.env.template` to `.env`
3. Change permissions with `chmod 600 .env`
4. Complete the file
5. Create a symbolic link to `.env` using `sudo ln -s /<ABSOLUTE_PATH>/.env /opt/docker-env`

### Systemd Mailjet
Send mail on failed unit. See https://github.com/JensWillemsens/systemd-mailjet for more info.
1. Create a new user with `sudo useradd -r systemd-mailjet`
2. Copy executable `bin/systemd-mailjet` to `/opt/bin/systemd-mailjet`
3. Make executable using `sudo chmod +x /opt/bin/systemd-mailjet`
4. Complete file `conf/systemd-mailjet.conf` and copy to `/opt/conf/systemd-mailjet.conf`
5. Make file readonly by owner `sudo chmod 400 /opt/conf/systemd-mailjet.conf`
6. Set correct owner `sudo chown systemd-mailjet:systemd-mailjet /opt/conf/systemd-mailjet.conf`

### Backup Nextcloud
1. Copy script `bin/backup-nextcloud` to `/opt/bin/backup-nextcloud` and make executable
2. Edit script and add correct information below heading OPTIONS
3. Set correct permissions `sudo chmod 500 /opt/bin/backup-nextcloud`

### BTRFS snapshots
1. Download the [BTRFS snap script](https://github.com/jf647/btrfs-snap) to `/opt/bin/btrfs-snap` and make executable
2. Copy script `bin/btrfs-snapshot` to `/opt/bin/btrfs-snapshot` and make executable
3. Set correct permissions `sudo chmod 544 /opt/bin/btrfs-snap /opt/bin/btrfs-snapshot`

### Containers
#### Before up

##### Traefik
Edit file `conf/traefik.toml` and change parameters `domain` and `email`

##### NC-Elasticsearch
Copy config `conf/sysctl.d/50-max-map-count.conf` to `/etc/sysctl.d/50-max-map-count.conf`

#### After up
##### Borgmatic
1. Execute `docker exec -it borgmatic sh -c "ssh <REMOTE_USER>@<REMOTE_URL>"`, check and accept the host key
2. Execute `ssh-keygen` and create a new ssh key with blank passphrase in `config/borgmatic/ssh`
3. Add public key to allowed ssh keys at remote host (depending on service)
4. Copy from template and edit `config/borgmatic/borgmatic.d/config.yaml`
5. Init repo if required with `docker exec borgmatic sh -c "borgmatic --init --encryption repokey-blake2"`

##### Nextcloud
Disable following apps:
- First run wizard 

Install and configure following apps:
- OnlyOffice
- Calendar
- Deck
- Group folders
- Notes
- Polls
- Quota warning
- Tasks
- Full text search 
- Full text search - Elasticsearch Platform
- Full text search - Files
- Preview generator
- Unsplash

##### Deluge
1. Go through online settings
  - Category Downloads
    - Set "Download to" to `/running`
    - Set "Move completed to" to `/downloads`
    - Set "Allocation" to "Use Full"
  - Category Network
    - Change incoming port to fixed port `58946`
  - Category Interface
    - Change password
  - Category Queue
    - Set "Share ratio limit" to 2
    - Enable "Stop seeding when share ratio reaches", set to 2
    - Enable "Remove torrent when share ratio is reached"
2. Stop container
3. Edit web.conf
  - Set "default_daemon" to `127.0.0.1:58846`
4. Start container

#### Plex
Go to https://app.plex.tv to setup following libraries:
- Films
  - /data/media/Movies
  - /data/optimized/movies
  - /data/media/Nazien
- TV Series
  - /data/media/TV Shows
  - /data/optimized/shows
  - /data/media/Nazien
- Foto's
  - /data/media/Photos
- Muziek
  - /data/media/Music

## Scheduled jobs
### One shot
- 5 min after boot: Nextcloud FullTextSearch index job (olaf-clc.yml => nextcloud-fulltextsearch-index.timer)

### Continuous
- Every 5 mins: Nextcloud cron.php (olaf-clc.yml => nextcloud-cron.timer)
- Every 10 mins: Nextcloud generate previews (olaf-clc.yml => nextcloud-preview-generator.timer)
- Every 15 mins: Update IP in DNS (olaf-clc.yml => cloudflare-dyndns.timer)
- Every hour: Take BTRFS snapshot (olaf-clc.yml => btrfs-snapshot.timer)

### 01:00 Daily application jobs
- None

### 02:00 Prepare backup
- Dump Nextcloud DB (olaf-clc.yml => dump-nc-db.timer)
- Dump Nextcloud calendars and contacts (olaf-clc.yml => backup-nc-calcardbackup.timer)

### 03:00 Perform backup
- Run Borgmatic (config/borgmatic/borgmatic.d/crontab.txt)

### 04:00 Perform application updates
- Run Watchtower (docker-compose.yml)

### 05:00 System tasks
- Update and restart (locksmith)
