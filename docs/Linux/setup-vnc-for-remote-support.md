# VNC Server Setup Guide for Linux Mint

Guide for setting up remote desktop support on a new Linux Mint installation. This allows you to SSH into the machine via Tailscale and start a VNC server to remotely support the user.

## Use Case

You've set up a new Linux Mint machine for someone and need to provide remote support. The machine has Tailscale installed for secure remote access. You'll SSH in as your admin user and start a VNC server running as the user's account so you can see and control their desktop environment.

**Important:** This method uses `sudo` to run commands as the target user, so you don't need to know their password - you only need sudo privileges on your SSH account.

## Prerequisites

- Tailscale installed and configured on the Linux Mint machine
- SSH access to the Linux Mint server (via Tailscale)
- Sudo privileges on your SSH user account
- Know which user account needs remote desktop support

## Installation

### 1. Install TigerVNC Server

```bash
sudo apt update
sudo apt install tigervnc-standalone-server tigervnc-common
```

## Setup for a Different User

If you need to set up VNC for a user different from your SSH login user:

### 2. Set VNC Password for Target User

```bash
sudo -u username vncpasswd
```

Enter a password when prompted (this is the VNC connection password, not the user's login password).

### 3. Create VNC Startup Configuration

Create the xstartup script to launch the desktop environment:

```bash
sudo -u username bash -c 'cat > /home/username/.vnc/xstartup << EOF
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec cinnamon-session
EOF'
```

Make it executable:

```bash
sudo chmod +x /home/username/.vnc/xstartup
```

### 4. Configure VNC to Listen on All IP Addresses

Create the VNC config file:

```bash
sudo -u username bash -c 'mkdir -p /home/username/.vnc'
sudo -u username bash -c 'echo "localhost=no" > /home/username/.vnc/config'
```

## Starting the VNC Server

### Start VNC Server

```bash
sudo -u username vncserver :1
```

The `:1` means display 1 (port 5901). You can use `:2` for port 5902, etc.

### Verify VNC is Running

Check if VNC is listening on all interfaces:

```bash
sudo ss -tulpn | grep 5901
```

You should see `0.0.0.0:5901` (not `127.0.0.1:5901`).

## Managing VNC Server

### Stop VNC Server

```bash
sudo -u username vncserver -kill :1
```

### List Running VNC Sessions

```bash
sudo -u username vncserver -list
```

### Restart VNC Server

```bash
sudo -u username vncserver -kill :1
sudo -u username vncserver :1
```

## Connecting to VNC

- **Host:** `server-ip-address:5901` (or just `:1` if using display number)
- **Password:** The password you set with `vncpasswd`

## Firewall Configuration

If you have a firewall enabled, allow VNC port:

```bash
sudo ufw allow 5901/tcp
```

## Alternative Desktop Environments

If Cinnamon doesn't work, edit the xstartup script to use a different desktop:

**For MATE:**
```bash
exec mate-session
```

**For XFCE:**
```bash
exec startxfce4
```

**Simple test with xterm:**
```bash
sudo -u username vncserver -xstartup /usr/bin/xterm :1
```

## Security Recommendations

**Option 1: Use SSH Tunneling (Recommended)**

Instead of exposing VNC directly, tunnel through SSH:

```bash
ssh -L 5901:localhost:5901 username@server-ip
```

Then connect VNC client to `localhost:5901`. This keeps VNC encrypted and secure.

**Option 2: Restrict to Specific IPs**

If you must expose VNC directly, use firewall rules to limit access:

```bash
sudo ufw allow from trusted-ip-address to any port 5901
```

## Troubleshooting

### Session Exits with Status 1

Check the log file:
```bash
cat /home/username/.vnc/hostname:1.log
```

Usually means the xstartup script needs to be fixed (see Alternative Desktop Environments section).

### Grey Screen on Connection

Desktop environment not starting properly. Check xstartup configuration.

### Cannot Connect from Remote

- Verify VNC is listening on all IPs: `sudo ss -tulpn | grep 5901`
- Check firewall: `sudo ufw status`
- Verify VNC is running: `sudo -u username vncserver -list`

## Quick Reference

```bash
# Start VNC
sudo -u username vncserver :1

# Stop VNC
sudo -u username vncserver -kill :1

# Check status
sudo ss -tulpn | grep 590

# View logs
cat /home/username/.vnc/*.log
```

## Notes

- Replace `username` with the actual username
- Display `:1` uses port `5901`, `:2` uses `5902`, etc.
- VNC password is stored in `/home/username/.vnc/passwd`
- Each user can run their own VNC session
