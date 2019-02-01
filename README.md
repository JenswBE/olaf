# Olaf the Great NAS
Config for my home NAS

## CloudLinux Config
Change following settings in the config:
- User settings (name, password and home folder)
- CloudFlare DynDNS (email, token, zone name and domain name)
- VPS backup user
- Mounts
- ...

## Post installation

### Install binaries

#### Systemd Mailjet
Send mail on failed unit. See https://github.com/JensWillemsens/systemd-mailjet for more info.
1. Create a new user with `sudo useradd -r systemd-mailjet`
2. Copy executable `bin/systemd-mailjet` to `/opt/bin/systemd-mailjet` and make executable
3. Create a new folder `sudo mkdir /opt/conf`
4. Complete file `conf/systemd-mailjet.conf` and copy to `/opt/conf/systemd-mailjet.conf`
5. Make file readonly by owner `sudo chmod 400 /opt/conf/systemd-mailjet.conf`
6. Set correct owner `sudo chown systemd-mailjet:systemd-mailjet /opt/conf/systemd-mailjet.conf`

#### Backup Nextcloud
1. Copy script `bin/backup-nextcloud` to `/opt/bin/backup-nextcloud` and make executable
2. Edit script and add correct information below heading OPTIONS
3. Set correct permissions `sudo chmod 500 /opt/bin/backup-nextcloud`

#### Backup VPS
1. Setup receiver
  1. Create a new user with `sudo useradd -r <RECV_USER>`
  2. Create new SSH key pair with `sudo -u <RECV_USER> ssh-keygen`
  3. Send the public key `~/.ssh/id_rsa.pub` to the sender through SCP
2. Setup sender
  1. Create a new user with `sudo useradd -rR /var/www/html <BACKUP_USER>`
  2. Add new user to ssh allowed group (if setup) with `sudo usermod -a -G ssh_users <BACKUP_USER>`
  3. Create new SSH key pair with `sudo -u <BACKUP_USER> ssh-keygen`
  4. Append the public key of step 1.3 (RECEIVER) to the file `~/.ssh/authorized_keys`
  5. Create a new SQL user with `GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO '<SQL_USER>'@'localhost' IDENTIFIED BY '<SQL_PASS>';`
  6. Flush privileges with `FLUSH PRIVILEGES;`
  7. Test if you can dump the database with `mysqldump -A -u <SQL_USER> -p<SQL_PASS>`
3. Verify on the RECEIVER if you can connect without password through SSH
4. Copy script `bin/backup-vps` to `/opt/bin/backup-vps` and make executable
5. Edit script and add correct information below heading OPTIONS
6. Set correct permissions `sudo chmod 500 /opt/bin/backup-vps`

#### Docker Compose
Use [following instructions](https://docs.docker.com/compose/install/#install-compose) and install Docker compose at `/opt/bin/docker-compose`

#### BTRFS snapshots
1. Download the [BTRFS snap script](https://github.com/jf647/btrfs-snap) to `/opt/bin/btrfs-snap` and make executable
2. Copy script `bin/btrfs-snapshot` to `/opt/bin/btrfs-snapshot` and make executable
3. Set correct permissions `sudo chmod 544 /opt/bin/btrfs-snap /opt/bin/btrfs-snapshot`

### Setup containers

#### Traefik
Edit file `conf/traefik.toml` and change parameters `domain` and `email`

#### Nextcloud
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

#### NC-Elasticsearch
Copy config `conf/sysctl.d/50-max-map-count.conf` to `/etc/sysctl.d/50-max-map-count.conf`

#### Deluge
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
