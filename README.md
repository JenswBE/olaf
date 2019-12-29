# Olaf the Great NAS
Config for my home NAS

## Setup
### Install CoreOS
1. Download the latest version of the [config transpiler](https://github.com/coreos/container-linux-config-transpiler/)
2. Complete config `olaf-clc.yml`. Use `mkpasswd --method=SHA-512 --rounds=4096` to generate a secure password hash.
3. Transpile the config using `ct -strict < olaf-clc.yml > olaf-clc.json`
4. Install CoreOS using `coreos-install -d /dev/sdX -i olaf-clc.json`
5. Reboot
6. Change hostname with `sudo hostnamectl set-hostname olaf`
7. Set correct timezone with `sudo timedatectl set-timezone Europe/Brussels`
8. Create opt folders with `sudo mkdir -p /opt/bin /opt/conf`

### Docker Compose
Use [following instructions](https://docs.docker.com/compose/install/#install-compose) and install Docker compose at `/opt/bin/docker-compose`

### Basic setup
1. Clone this repo with `git clone https://github.com/JenswBE/olaf.git`
2. Copy file `.env.template` to `.env`
3. Change permissions with `chmod 600 .env`
4. Complete the file
5. Create a symbolic link to `.env` using `sudo ln -s /<ABSOLUTE_PATH>/.env /opt/docker-env`

### Systemd Mailjet
Send mail on failed unit. See [JenswBE/systemd-mailjet](https://github.com/JenswBE/systemd-mailjet) for more info.
1. Create a new user with `sudo useradd -r systemd-mailjet`
2. Copy executable `bin/systemd-mailjet` to `/opt/bin/systemd-mailjet`
3. Make executable using `sudo chmod +x /opt/bin/systemd-mailjet`
4. Complete file `conf/systemd-mailjet.conf` and copy to `/opt/conf/systemd-mailjet.conf`
5. Make file readonly by owner `sudo chmod 400 /opt/conf/systemd-mailjet.conf`
6. Set correct owner `sudo chown systemd-mailjet:systemd-mailjet /opt/conf/systemd-mailjet.conf`

### Containers
#### After up
##### Borgmatic
1. Execute `docker exec -it borgmatic sh -c "ssh -p <PORT> <BORG_USER>@<BORG_HOST>"`, check and accept the host key
2. Execute `ssh-keygen` and create a new ssh key with blank passphrase in `conf/borgmatic/ssh`
3. Add public key to allowed ssh keys at remote host (depending on service)
4. Copy from template and edit `conf/borgmatic/borgmatic.d/config.yaml`
5. Change permissions with `chmod 600 config.yaml`
6. Init repo if required with `docker exec borgmatic sh -c "borgmatic --init --encryption repokey-blake2"`
7. Perform a backup to test the setup with `docker exec borgmatic sh -c "borgmatic --verbosity 1"`
8. Optional: Backup your repo key file with `docker exec borgmatic sh -c "BORG_RSH=\"ssh -i /root/.ssh/<NAME_OF_SSH_KEY>\" borg key export --qr-html <FULL_REPO_NAME> /root/.ssh/repokey.html"`. Your file is available at `conf/borgmatic/ssh/repokey.html`.

##### Transmission
1. Set correct permission with `sudo chown 233:233 /media/data/services/transmission/`
2. Go through online settings
  - Category Torrents
    - Set "Download to" to `/downloads`
    - Set "Directory for incomplete files" to `/running`
    - Set "Stop seeding at ratio" to 3
  - Category Queue
    - Set "Download Queue Size" to 10

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

#### MinIO
You can create a new bucket and assign read/write right to a user with following commands:
1. Start the MinIO client: `docker run -it --entrypoint=/bin/sh minio/mc`
2. Add a new server: `mc config host add remote <URL> <ACCESS_KEY> <SECRET_KEY>`
3. Set bucket name for easy reference: `BUCKET=<REPLACE_ME>`
4. Create bucket: `mc mb remote/${BUCKET:?}`
5. Create readwrite policy:
```bash
cat > ${BUCKET:?}-rw.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": ["s3:*"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::${BUCKET:?}/*"]
    }
  ]
}
EOF
```
6. Add policy to MinIO server: `mc admin policy add remote ${BUCKET:?}-rw ${BUCKET:?}-rw.json`
7. Create new user: `mc admin user add remote <USERNAME> <PASSWORD>`
8. Assign policy to user: `mc admin policy set remote ${BUCKET:?}-rw <USERNAME>`

## Scheduled jobs
### One shot
- None

### Continuous
- Every 15 mins: Update IP in DNS (olaf-clc.yml => cloudflare-dyndns.timer)
- Every hour: Take BTRFS snapshot (olaf-clc.yml => btrfs-snapshot.timer)

### 01:00 Daily application jobs
- None

### 02:00 Prepare backup
- None

### 03:00 Perform backup
- Run Borgmatic (conf/borgmatic/borgmatic.d/crontab.txt)

### 04:00 Perform application updates
- Run Watchtower (docker-compose.yml)

### System tasks
- 05:00 Update and restart (locksmith)
- 06:00 First of month: Scrub BTRFS filesystem (olaf-clc.yml => btrfs-scrub.timer)
