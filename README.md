# Olaf the Great NAS
Config for my home NAS

## Post installation

### Deluge
1. Go through online settings
2. Stop container
3. Edit web.conf
  - Set "interface" to "192.168.1.250"
  - Set "default_daemon" to "127.0.0.1:58846"
4. Start container
