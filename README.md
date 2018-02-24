# Olaf the Great NAS
Config for my home NAS

## Post installation

### Letsencrypt
1. Edit file "/opt/appdata/letsencrypt/nginx/site-confs/default"
2. Create "server"-block for following domains:
  - "torrent.*" => "http://192.168.1.250:8112"
  - "nextcloud.*" => http://nextcloud

### Deluge
1. Go through online settings
  - Category Downloads
    - Set "Download to" to "/running"
    - Set "Move completed to" to "/downloads"
    - Set "Allocation" to "Use Full"
  - Category Interface
    - Change password
  - Category Queue
    - Set "Share ratio limit" to 2
    - Enable "Stop seeding when share ratio reaches", set to 2
    - Enable "Remove torrent when share ratio is reached"
2. Stop container
3. Edit web.conf
  - Set "interface" to "192.168.1.250"
  - Set "default_daemon" to "127.0.0.1:58846"
4. Start container

### Plex
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
