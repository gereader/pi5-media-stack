# Pi 5 Media Stack

An Ansible powered setup for running a small media stack on a Raspberry Pi 5:

- qBittorrent with Web UI  
- Gluetun VPN container configured for Private Internet Access (PIA)  
- Audiobookshelf for audiobooks and podcasts  
- All services run in Docker, configured with Ansible  

The goal is to have a reproducible, documented, and shareable way to deploy this on a Pi, not to do anything sketchy. It started with wanting a better way to self-host audiobooks and grew into a small modular stack that runs well on a Raspberry Pi.

## Legal and ethical use

This project is intended only for lawful personal use.

While it includes a BitTorrent client (qBittorrent), it is designed to:

- download Linux ISOs, public domain media, and other legally distributable files  
- demonstrate containerization, VPN routing, and infrastructure as code with Ansible  
- provide a reproducible home lab deployment pattern for self-hosted services  

You must only download content you are legally permitted to access.  
I do not condone or encourage piracy of any kind.  
The responsibility for lawful use falls entirely on the user.

## Architecture

- Raspberry Pi 5 running Raspberry Pi OS  
- External USB drive mounted at `/mnt/media` (ext4)  
- Docker & Docker Compose installed on the Pi  
- Ansible playbook (run from macOS) that:
  - installs Docker  
  - mounts the media drive  
  - creates media directories  
  - renders a Docker Compose file  
  - deploys the containers  

## Repository layout

```text
ansible/
  group_vars/
    all.yml               # global defaults
    pi_media.yml          # host specific vars
    pi_media.vault.yml    # encrypted VPN secrets
  inventory.ini
  roles/
  site.yml
  templates/
    docker-compose.yml.j2
LICENSE
README.md
```

## Requirements

Target machine (Raspberry Pi 5):

- ext4 drive mounted at `/mnt/media`
- SSH enabled

Control machine (macOS):

- Python 3.10+
- Ansible  
- Docker Python SDK  
- PIA account with OpenVPN credentials  

## Development environment (macOS with uv)

```
curl -LsSf https://astral.sh/uv/install.sh | sh

cd pi5-media-stack
uv venv
source .venv/bin/activate

uv pip install ansible docker
ansible-galaxy collection install community.docker
```

## Setup

### 1. Clone the repo

```
git clone https://github.com/YOURNAME/pi5-media-stack.git
cd pi5-media-stack
```

### 2. Edit the inventory

```
[pi_media]
pi5 ansible_host=192.168.1.50 ansible_user=gene
```

### 3. Review defaults

`ansible/group_vars/all.yml` contains defaults for ports, directories, UID/GID, etc.

### 4. Add PIA credentials to the vault

```
ansible-vault edit ansible/group_vars/pi_media.vault.yml
```

Example content:

```
vault_pia_username: "pxxxxxxx"
vault_pia_password: "yourpassword"
```

### 5. Run the playbook

```
ansible-playbook -i ansible/inventory.ini ansible/site.yml --ask-vault-pass
```

---

# Accessing the services

- qBittorrent Web UI:  
  `http://PI_ADDRESS:8080`

- Audiobookshelf:  
  `http://PI_ADDRESS:13378`

Audiobookshelf scans from:

- `/mnt/media/audiobooks`

---

# qBittorrent first-time login

On first boot, qBittorrent generates a temporary admin password.  
Retrieve it with:

```
docker logs qbittorrent | grep -i password -A2
```

You will see:
```
The WebUI administrator username is: admin
The WebUI administrator password was not set. A temporary password is provided for this session: SomePassword
```
### Default login

- **Username:** admin  
- **Password:** (from logs)

### Change the password

1. Open WebUI → Tools → Options  
2. Select "WebUI"  
3. Change admin password  
4. Save  

Password will persist inside the `/config` volume.

---

# Verifying VPN routing

From inside Gluetun:

```
docker exec gluetun sh -c "apk add --no-cache curl >/dev/null 2>&1 || true; curl https://ifconfig.me"
```

Should **not** match your home IP.  
Should be something like:
```
173.244.56.72
```

From **host machine**, your home IP will still show normally:

```
curl https://ifconfig.me
```

---

# Full deployment flow

After filling:

- `inventory.ini`  
- Vault secrets  

Run:

```
ansible-playbook -i ansible/inventory.ini ansible/site.yml --ask-vault-pass
```

After deployment, verify:

```
docker exec gluetun curl https://ifconfig.me
```


