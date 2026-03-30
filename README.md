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

The tunnel must be started separately before mounting. Tunnel management is intentionally decoupled from mount/unmount so that authentication (which requires a browser) can be handled cleanly.

### Starting the tunnel

```bash
./start-cf-tunnel myserverCf
```

The script starts `cloudflared` in the background. If a CF Access login is required, it will print the URL:

```
--------------------------------------------------------------
 Login required. Open this URL in your browser:

  https://yourteam.cloudflareaccess.com/cdn-cgi/access/cli?...

 Waiting for authentication (up to 2 minutes)...
--------------------------------------------------------------
```

Once authenticated, the tunnel runs silently and the script exits. The PID is saved to `/tmp/cf-tunnel-<name>.pid`.

### Mounting after the tunnel is up

```bash
sudo ./mount-network myserverCf
```

`mount-network` will error out with a clear message if the tunnel isn't running yet.

### Stopping the tunnel

```bash
./stop-cf-tunnel myserverCf
```

This reads the saved PID and kills the `cloudflared` process. The tunnel is independent of the mount — unmounting does not stop it.

### Install `cloudflared`

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
| `start-cf-tunnel` | Starts a Cloudflare tunnel, handles CF Access login |
| `stop-cf-tunnel` | Stops a running Cloudflare tunnel |
