+++
title = "How to install PeerTube on Ubuntu" 
+++

PeerTube is a self-hosted video streaming platform, with various social networking features that I am not using. I need this to host training videos for SENAITE. 

I first installed it using [caprover/caprover: Scalable PaaS (automated Docker+nginx) - aka Heroku on Steroids](https://github.com/caprover/caprover) which was pretty easy, but later took down that VM to reduce my [Günstige Dedicated Server, Cloud & Hosting aus Deutschland](https://www.hetzner.com/) costs. The following instructions are for installing PeerTube on Ubuntu 24.04. 

I first created an Ubuntu LXC on Proxmox on my home server. I am mainly using free [Cloudflare Tunnels](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/) to expose self-hosted web services to the public Internet, and Cloudflare doesn't like you using a lot of bandwidth for these, but I don't think it will be high volume.

I gave the instance 4GB RAM and 30GB storage. It got an IP address from DHCP. 

Log into the instance. The following code installs Docker and other software required, then creates a user and then switches to being that user for the subsequent steps.

```bash
apt update
apt install apt-transport-https vim ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt-cache policy docker-ce
apt install docker-ce
systemctl enable docker
systemctl status docker
useradd -m paul -s /bin/bash
passwd paul  # use a strong password
usermod -aG sudo paul
usermod -aG docker paul
su - paul
```

Then I create a folder to keep things tidy and in that folder I create two files: `docker-compose.yml` and `.env`. The content of these files will be different if you are not using  Cloudflare Tunnels - this stage repeatedly caught me out, but this is the configuration that ultimately worked for me. 

```bash
mkdir docker
cd docker
vim docker-compose.yml
vim .env
```

- `docker-compose.yml`: use your own secure password for the database (this password should be the same in both config files)

```
version: '3.7'
services:
  peertube:
    image: chocobozzz/peertube:production-bookworm
    container_name: peertube
    ports:
    - "9000:9000"
    volumes:
    - ./data/config:/config
    - ./data/storage:/data
    - ./data/logs:/logs
    env_file:
    - .env
    depends_on:
    - postgres
    - redis
    restart: unless-stopped
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: peertube
      POSTGRES_PASSWORD: xxxxxxxxxxxxxx
      POSTGRES_DB: peertube
    volumes:
    - ./data/db:/var/lib/postgresql/data
    restart: unless-stopped
  redis:
    image: redis:6
    restart: unless-stopped
```
	
- `.env`: add your database password and webserver hostname; add the IP address of the container/VM to the trusted proxies; create a secret using the suggested command; add your email address
	
```
	# Database / Postgres service configuration
POSTGRES_USER=peertube
POSTGRES_PASSWORD=xxxxxxxxxxxxxx
# Postgres database name "peertube"
POSTGRES_DB=peertube
# The database name used by PeerTube will be PEERTUBE_DB_NAME (only if set) *OR* 'peertube'+PEERTUBE_DB_SUFFIX
#PEERTUBE_DB_NAME=<MY POSTGRES DB NAME>
#PEERTUBE_DB_SUFFIX=_prod
# Database username and password used by PeerTube must match Postgres', so they are copied:
PEERTUBE_DB_USERNAME=$POSTGRES_USER
PEERTUBE_DB_PASSWORD=$POSTGRES_PASSWORD
PEERTUBE_DB_SSL=false
# Default to Postgres service name "postgres" in docker-compose.yml
PEERTUBE_DB_HOSTNAME=postgres

# PeerTube server configuration
# If you test PeerTube in local: use "peertube.localhost" and add this domain to your host file resolving on 127.0.0.1
PEERTUBE_WEBSERVER_HOSTNAME=xxxxxxxx.xxxxxxxxxx.xxx
# If you just want to test PeerTube on local
#PEERTUBE_WEBSERVER_PORT=9000
#PEERTUBE_WEBSERVER_HTTPS=false
# If you need more than one IP as trust_proxy
# pass them as a comma separated array:
PEERTUBE_TRUST_PROXY=["127.0.0.1", "loopback", "xxx.xxx.x.xxx/24"]

# Generate one using `openssl rand -hex 32`
PEERTUBE_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# E-mail configuration
# If you use a Custom SMTP server
#PEERTUBE_SMTP_USERNAME=
#PEERTUBE_SMTP_PASSWORD=
# Default to Postfix service name "postfix" in docker-compose.yml
# May be the hostname of your Custom SMTP server
# PEERTUBE_SMTP_HOSTNAME=postfix
# PEERTUBE_SMTP_PORT=25
# PEERTUBE_SMTP_FROM=noreply@<MY DOMAIN>
# PEERTUBE_SMTP_TLS=false
# PEERTUBE_SMTP_DISABLE_STARTTLS=false
PEERTUBE_ADMIN_EMAIL=xxxxxxx@xxxxxxxxxx.xxx

# Postfix service configuration
# POSTFIX_myhostname=<MY DOMAIN>
# If you need to generate a list of sub/DOMAIN keys
# pass them as a whitespace separated string <DOMAIN>=<selector>
# OPENDKIM_DOMAINS=<MY DOMAIN>=peertube
# see https://github.com/wader/postfix-relay/pull/18
# OPENDKIM_RequireSafeKeys=no

# If you want to enable object storage for PeerTube, set the following variables.
#PEERTUBE_OBJECT_STORAGE_ENABLED=
#PEERTUBE_OBJECT_STORAGE_ENDPOINT=
#PEERTUBE_OBJECT_STORAGE_REGION=
#PEERTUBE_OBJECT_STORAGE_CREDENTIALS_ACCESS_KEY_ID=
#PEERTUBE_OBJECT_STORAGE_CREDENTIALS_SECRET_ACCESS_KEY=
#PEERTUBE_OBJECT_STORAGE_STREAMING_PLAYLISTS_BUCKET_NAME=
#PEERTUBE_OBJECT_STORAGE_STREAMING_PLAYLISTS_PREFIX=
#PEERTUBE_OBJECT_STORAGE_STREAMING_PLAYLISTS_BASE_URL=
#PEERTUBE_OBJECT_STORAGE_WEB_VIDEOS_BUCKET_NAME=
#PEERTUBE_OBJECT_STORAGE_WEB_VIDEOS_PREFIX=
#PEERTUBE_OBJECT_STORAGE_WEB_VIDEOS_BASE_URL=
#PEERTUBE_OBJECT_STORAGE_USER_EXPORTS_BUCKET_NAME=
#PEERTUBE_OBJECT_STORAGE_USER_EXPORTS_PREFIX=
#PEERTUBE_OBJECT_STORAGE_USER_EXPORTS_BASE_URL=
#PEERTUBE_OBJECT_STORAGE_ORIGINAL_VIDEO_FILES_BUCKET_NAME=
#PEERTUBE_OBJECT_STORAGE_ORIGINAL_VIDEO_FILES_PREFIX=
#PEERTUBE_OBJECT_STORAGE_ORIGINAL_VIDEO_FILES_BASE_URL=
#PEERTUBE_OBJECT_STORAGE_CAPTIONS_BUCKET_NAME=
#PEERTUBE_OBJECT_STORAGE_CAPTIONS_PREFIX=
#PEERTUBE_OBJECT_STORAGE_CAPTIONS_BASE_URL=

# Comment these variables if your S3 provider does not support object ACL
PEERTUBE_OBJECT_STORAGE_UPLOAD_ACL_PUBLIC="public-read"
PEERTUBE_OBJECT_STORAGE_UPLOAD_ACL_PRIVATE="private"

#PEERTUBE_LOG_LEVEL=info

# /!\ Prefer to use the PeerTube admin interface to set the following configurations /!\
#PEERTUBE_SIGNUP_ENABLED=true
#PEERTUBE_TRANSCODING_ENABLED=true
#PEERTUBE_CONTACT_FORM_ENABLED=true
```

Then fire up the containers:

```bash
docker compose up -d
```

If it is not working you can check the Docker logs for clues to what you have done wrong. Look out for the web login details in the Docker output. 

I then pointed a Cloudflare Tunnel at `http://<IP address>:9000` - job done. 

