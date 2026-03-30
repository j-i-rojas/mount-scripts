# mount-scripts

Bash scripts to mount and unmount CIFS/SMB network shares, with optional Cloudflare Tunnel support.

## Requirements

- `cifs-utils` — for mounting SMB shares
- `cloudflared` — only needed for Cloudflare Tunnel mounts

```bash
sudo apt install cifs-utils
```

## Setup

### 1. Configure your mounts

Copy the example config and fill in your values:

```bash
cp .env.example .env
```

Edit `.env` with your mount definitions:

```bash
# Format: MOUNT_name="server_ip|share|mount_point|domain|username[|port][|cf_tunnel_hostname]"

MOUNT_myserver="192.168.1.10|C$|/mnt/myserver|MYDOMAIN|administrator"
MOUNT_nas="192.168.1.20|shared|/mnt/nas|WORKGROUP|user"

# Credentials file path
CREDS_FILE="/home/youruser/.smb_credentials"
```

The `.env` file is gitignored — your IPs, hostnames, and credentials stay off the repo.

### 2. Set up a credentials file (recommended)

Create `~/.smb_credentials` so you don't have to type the password each time:

```
username=administrator
password=yourpassword
domain=MYDOMAIN
```

Secure it:

```bash
chmod 600 ~/.smb_credentials
```

If `CREDS_FILE` is not set or the file doesn't exist, the script will prompt for a password instead.

### 3. Make the scripts executable

```bash
chmod +x mount-network umount-network
```

## Usage

### Mount

```bash
sudo ./mount-network <name>
```

### Unmount

```bash
sudo ./umount-network <name>
# Keep the Cloudflare tunnel running after unmounting:
sudo ./umount-network <name> --keep-tunnel
```

Running either script with no arguments lists all configured mounts.

## Cloudflare Tunnel mounts

For mounts that go through a Cloudflare Tunnel, set the server IP to `127.0.0.1` and add the port and tunnel hostname as the last two fields:

```bash
MOUNT_myserverCf="127.0.0.1|C$|/mnt/myserver|MYDOMAIN|administrator|4450|smb.example.com"
```

The script will:
1. Check if `cloudflared` is installed.
2. Start the tunnel automatically (`cloudflared access tcp --hostname smb.example.com --url localhost:4450`).
3. Prompt for browser authentication if needed.
4. Stop the tunnel on unmount (unless `--keep-tunnel` is passed or another mount is still using it).

Install `cloudflared` if needed:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

## File reference

| File | Description |
|------|-------------|
| `.env` | Your mount definitions — **gitignored, do not commit** |
| `.env.example` | Template to copy from — safe to commit |
| `mount-network` | Mounts a configured share |
| `umount-network` | Unmounts a configured share |
