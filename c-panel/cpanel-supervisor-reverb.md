# Laravel Reverb & Supervisor Setup Guide on cPanel (Shared Hosting)

This guide documents the complete step-by-step process of installing and running **Laravel Reverb (WebSocket Server)** using a local **Supervisor** installation on a shared cPanel hosting environment without root/sudo privileges.

---

## 📌 Problem Statement
The official Laravel Reverb documentation assumes you have **root access** to a VPS/Dedicated server (`apt install supervisor`, access to `/etc/supervisor/`, and binding to global ports like `6001`). On **Shared Hosting**:
* `sudo` commands return: `sudo: effective uid is not 0... (nosuid option set)`.
* Default WebSocket ports like `6001` or standard ports like `8080` are often already in use (`Address already in use - EADDRINUSE`) by other users on the shared server.
* Background daemons get killed automatically due to strict memory/file descriptor policies (`minfds`).

---

## 🛠️ Step-by-Step Implementation Solution

### Step 1: Install Supervisor Locally (Without Root)
Since you cannot install Supervisor system-wide, install it inside your user space using Python's package manager (`pip`):

```bash
pip install --user supervisor
```

### Step 2: Generate and Configure `supervisord.conf`
Generate a fresh user-level configuration template in your user home directory:

```bash
~/.local/bin/echo_supervisord_conf > ~/supervisord.conf
```

Open the file for editing using `nano ~/supervisord.conf` or update it via terminal commands. Due to shared hosting restrictions, you **must lower the system limit constraints** (`minfds` and `minprocs`), otherwise Supervisor will fail to boot.

Execute these commands to safely lower the limits to compliant levels:
```bash
sed -i 's/^;*minfds=.*/minfds=10/' ~/supervisord.conf
sed -i 's/^;*minprocs=.*/minprocs=10/' ~/supervisord.conf
```

### Step 3: Append the WebSocket Program Block
Open `~/supervisord.conf`, scroll to the very bottom, and append your Laravel Reverb process block. 

*Note: Make sure to use an unassigned, unique custom port (e.g., `8099`) since shared ports like `6001` or `8080` are generally locked by other server environments.*

```ini
[program:websockets]
command=/usr/local/bin/php /home1/a17800cf/public_html/artisan reverb:start --host=127.0.0.1 --port=8099
numprocs=1
autostart=true
autorestart=true
user=a17800cf
redirect_stderr=true
stdout_logfile=/home1/a17800cf/public_html/storage/logs/reverb.log
```

### Step 4: Configure the Laravel `.env` File
Update your environment variables inside `/home1/a17800cf/public_html/.env` to separate the **Internal Socket Server Execution** from the **External Client Connections**:

```env
REVERB_APP_ID=10000000
REVERB_APP_KEY=drivemond
REVERB_APP_SECRET=drivemond

# Internal Daemon Configuration (Handled by Supervisor)
REVERB_SERVER_HOST=127.0.0.1
REVERB_SERVER_PORT=8099

# External Connection Variables (For Browsers / Mobile Clients via SSL Proxy)
REVERB_HOST=rapidpilot.in
REVERB_PORT=443
REVERB_SCHEME="https"
```

### Step 5: Clean Up Locks & Run the Daemon
Before starting, wipe out stale environment socket sockets or PID file footprints to prevent runtime communication blockages (`refused connection` errors):

```bash
rm -f /tmp/supervisor.sock* ~/supervisord.pid
```

Fire up the background Supervisor daemon:
```bash
~/.local/bin/supervisord -c ~/supervisord.conf
```

Check the process status:
```bash
~/.local/bin/supervisorctl -c ~/supervisord.conf status
```

### Step 6: Apache Reverse Proxy Setup (`.htaccess`)
Because external clients cannot access internal port `8099` due to firewall rules, create an Apache rewrite pipeline in your application's public root (`public_html/.htaccess`) to proxy incoming secure websocket traffic internally:

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    
    # Securely route external WebSocket connections to internal Reverb instance
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^(.*)\$ http://127.0.0\$1 [P,L]
</IfModule>
```

### Step 7: Automated Keep-Alive Cron Job
Shared hosting environments frequently terminate non-interactive user background tasks during server sweeps. To keep your Supervisor environment permanently alive, configure a cPanel Cron Job to execute **Every 5 Minutes**:

```bash
pgrep -u \$(whoami) -f supervisord > /dev/null || ~/.local/bin/supervisord -c ~/supervisord.conf
```

---
💡 *Derived from a live deployment debug session for `a17800cf@rapidpilot.in` hosting Laravel Reverb.*
