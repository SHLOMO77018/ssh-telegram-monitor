# SSH Telegram Monitor 🔐

![GitHub stars](https://img.shields.io/github/stars/SHLOMO77018/ssh-telegram-monitor)
![GitHub release](https://img.shields.io/github/v/release/SHLOMO77018/ssh-telegram-monitor)
![License](https://img.shields.io/github/license/SHLOMO77018/ssh-telegram-monitor)
![GitHub issues](https://img.shields.io/github/issues/SHLOMO77018/ssh-telegram-monitor)
![GitHub forks](https://img.shields.io/github/forks/SHLOMO77018/ssh-telegram-monitor)

Real-time SSH authentication monitoring and automated blocking system with Telegram notifications for Linux servers.

## Features 🚀

- **Real-time SSH Monitoring**: Instant notifications for successful and failed SSH login attempts
- **Auto-blocking**: Automatically blocks IPs after configurable failed attempts
- **Advanced User Management Interface**:
  - Full system user management via Telegram buttons
  - Create/delete users directly from Telegram
  - Per-user 2FA and Fail2Ban settings
  - SSH key management (add/remove/view)
- **Interactive Telegram UI (Pyrogram-based)**:
  - Lightning-fast button responses
  - Hebrew interface with intuitive navigation
  - Real-time dashboard with system stats
  - Service management controls
- **2FA Authentication System**:
  - PAM-integrated two-factor authentication
  - Per-user 2FA configuration
  - 30-second approval timeout
  - Instant session termination for denied logins
- **Telegram Group with Topics**:
  - Organized notifications in 5 different topics/threads
  - Separate channels for different event types
- **Multi-layer Security**:
  - Fail2ban integration
  - UFW firewall rules
  - iptables direct rules with tcp-reset
  - Active session termination
- **Smart Tracking**:
  - Database of blocked IPs
  - Failed attempts counter with time-based reset
  - Prevents duplicate notifications for blocked IPs

## Screenshots 📱

### Notification Examples:
- ✅ Successful login notification with action buttons
- ⚠️ Failed attempt warning (1/3, 2/3)
- 🚫 Auto-block notification after 3 attempts
- 🔓 Unblock confirmation

## Requirements 📋

- Ubuntu/Debian Linux
- Python 3.8+
- Root access
- Active Telegram bot

## Installation 🛠️

### 1. Clone the Repository
```bash
git clone https://github.com/SHLOMO77018/ssh-telegram-monitor.git
cd ssh-telegram-monitor
```

### 2. Install Dependencies
```bash
# System packages
apt update
apt install -y python3-pip fail2ban ufw conntrack python3-psutil

# Python packages
pip3 install requests psutil pyrogram tgcrypto python-telegram-bot python-dotenv
```

### 3. Create Telegram Bot
1. Open Telegram and search for @BotFather
2. Send `/newbot` and follow instructions
3. Save your bot token

### 4. Configure Environment
```bash
# Copy example config
cp .env.example .env

# Edit with your credentials
nano .env
# Add your BOT_TOKEN and run get_telegram_chat_id.py to get CHAT_ID
```

### 5. Run Setup Script
```bash
sudo ./install.sh
```

## Manual Installation 🔧

### 1. Copy Scripts
```bash
cp scripts/* /usr/local/bin/
chmod +x /usr/local/bin/*.sh
chmod +x /usr/local/bin/*.py
```

### 2. Configure PAM
Add to `/etc/pam.d/sshd`:
```bash
session    optional     pam_exec.so seteuid /usr/local/bin/ssh_login_notify.sh
```

### 3. Setup Systemd Services
```bash
cp systemd/*.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable ssh-telegram-monitor.service
systemctl enable telegram-action-handler.service
systemctl start ssh-telegram-monitor.service
systemctl start telegram-action-handler.service
```

### 4. Configure Fail2ban
```bash
cp config/jail.local.example /etc/fail2ban/jail.local
systemctl restart fail2ban
```

## Configuration ⚙️

### Environment Variables (.env)
```env
BOT_TOKEN=your_bot_token_here
CHAT_ID=your_chat_id_here
MAX_ATTEMPTS=3
BLOCK_DURATION_HOURS=24
```

### Get Your Chat ID
```bash
python3 scripts/get_telegram_chat_id.py
# Follow instructions and send /start to your bot
```

## Usage 📱

### Main Menu System
Send `/menu` to access the interactive control panel with:
- 🎛 **Dashboard** - Real-time system status and statistics
- 👥 **User Management** - Create/delete users, manage SSH keys
- 🔐 **Security Settings** - Configure 2FA and Fail2Ban
- 🚫 **Block Management** - View and unblock IPs
- 📊 **Statistics** - Detailed login and security stats
- ⚙️ **System Settings** - Restart services and system controls

### Telegram Commands
- `/menu` - Open main control panel
- `/adduser <username> [password]` - Create new system user
- `/addkey <username> <ssh-key>` - Add SSH public key
- `/block <IP>` - Manually block an IP
- `/unblock <IP>` - Unblock an IP
- `/enable2fa` - Enable global 2FA
- `/disable2fa` - Disable global 2FA
- `/status` - View system status

### User Management Features
- **Create/Delete Users**: Full system user management
- **SSH Key Management**: Add, remove, view SSH keys per user
- **Per-User 2FA**: Enable/disable 2FA for specific users
- **Per-User Fail2Ban**: Configure auto-blocking per user
- **Password Management**: Set/reset user passwords

### Interactive Buttons
Each notification includes:
- **✅ Approve / ❌ Deny** - For 2FA requests
- **🚫 Block IP** - Immediately block and disconnect
- **🔓 Unblock** - Remove all blocks
- **📊 More Info** - Get detailed information

### Service Management
```bash
# Check status
systemctl status ssh-telegram-monitor
systemctl status telegram-action-handler

# View logs
journalctl -u ssh-telegram-monitor -f
journalctl -u telegram-action-handler -f

# Restart services
systemctl restart ssh-telegram-monitor
systemctl restart telegram-action-handler
```

## File Structure 📁

```
ssh-telegram-monitor/
├── scripts/
│   ├── ssh_telegram_notify.py      # Login notifications
│   ├── ssh_monitor_advanced.py     # Failed attempts monitor
│   ├── telegram_action_handler.py  # Handle Telegram buttons
│   ├── get_telegram_chat_id.py     # Setup helper
│   ├── ssh_login_notify.sh         # PAM script
│   ├── kill_ssh_sessions.sh        # Session terminator
│   └── unblock_ip_complete.sh      # Unblock helper
├── systemd/
│   ├── ssh-telegram-monitor.service
│   └── telegram-action-handler.service
├── config/
│   ├── sshd.pam.example            # PAM configuration
│   └── jail.local.example          # Fail2ban config
├── docs/
│   └── setup_guide.md
├── .env.example
├── README.md
└── install.sh
```

## How It Works 🔍

1. **SSH Login Detection**: PAM module triggers on every SSH session
2. **Failed Attempts Tracking**: Monitors `/var/log/auth.log` for failures
3. **Smart Blocking**:
   - Counts attempts per IP
   - Auto-blocks after threshold
   - Maintains blocked IPs database
4. **Notifications**:
   - Sends detailed info via Telegram
   - Includes location, ISP, attempt count
   - Provides action buttons
5. **Multi-layer Blocking**:
   - Adds to Fail2ban
   - Creates UFW rule
   - Inserts iptables DROP
   - Kills active sessions

## Troubleshooting 🔨

### No Notifications
```bash
# Check services
systemctl status ssh-telegram-monitor
systemctl status telegram-action-handler

# Verify bot token
curl https://api.telegram.org/bot<YOUR_TOKEN>/getMe

# Check logs
tail -f /var/log/ssh_telegram.log
```

### False Positives
- Edit MAX_ATTEMPTS in .env
- Whitelist IPs in UFW
- Check `/var/run/ssh_blocked_ips.json`

### Clear All Blocks
```bash
/usr/local/bin/unblock_ip_complete.sh <IP>
```

## Security Considerations 🔒

- Keep bot token secret
- Use strong SSH keys
- Regular system updates
- Monitor logs regularly
- Implement rate limiting
- Use VPN for admin access

## Contributing 🤝

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Push to branch
5. Open Pull Request

## License 📄

MIT License - see LICENSE file

## Author 👤

Created with ❤️ for server security

## Support 💬

- Issues: [GitHub Issues](https://github.com/SHLOMO77018/ssh-telegram-monitor/issues)
- Discussions: [GitHub Discussions](https://github.com/SHLOMO77018/ssh-telegram-monitor/discussions)

## Changelog 📝

### v3.0.0 (Latest)
- **NEW: Complete UI Overhaul with Pyrogram**
  - Lightning-fast button responses (10x faster)
  - Full Hebrew interface
  - Comprehensive menu system
- **NEW: Advanced User Management**
  - Create/delete system users from Telegram
  - Manage SSH keys per user
  - Per-user 2FA and Fail2Ban settings
- **NEW: Interactive Dashboard**
  - Real-time system statistics
  - Service management controls
  - Blocked IPs management with unblock buttons
- **IMPROVED: 2FA System**
  - PAM integration for reliable authentication
  - Per-user configuration
  - Better session handling

### v2.0.0
- **NEW: Telegram Group Topics Support**
  - Organized notifications in 5 different topics/threads
  - Separate channels for successful logins, failed attempts, session ends, 2FA, and general alerts
- **NEW: 2FA Authentication System**
  - Real-time approval/deny for SSH logins via Telegram
  - 30-second timeout for 2FA requests
  - Instant session termination for denied logins
- **Improved: Enhanced callback handling**
  - Better button response handling
  - More detailed session information
- **Added: Session end notifications**
  - Track when users disconnect from SSH

### v1.0.0
- Initial release
- Basic monitoring and notifications
- Auto-blocking after 3 attempts
- Interactive Telegram buttons
- Multi-layer security implementation

## Setup Telegram Group with Topics 🏢

### Prerequisites for Group Features:
1. Create a Telegram group (not a channel)
2. Add your bot as administrator with these permissions:
   - Delete messages
   - Pin messages
   - Manage topics
3. **Enable Forums/Topics in group settings:**
   - Go to Group Info → Edit → Toggle "Topics" ON
4. Get your group ID (it will look like `-1003066710155`)

### Configure Group Features:
```bash
# Edit .env file
TELEGRAM_GROUP_ID=-1003066710155
2FA_ENABLED=true
2FA_TIMEOUT=30
```

### Initialize Topics:
After enabling forums in your group:
```bash
# Run in the group chat
/init
```

This will create 5 topics:
- ✅ **Successful Logins** - All successful SSH authentications
- ❌ **Failed Logins** - Failed attempts and auto-blocks
- 🚪 **Session End** - Notifications when SSH sessions end
- 🔐 **2FA Approval** - Two-factor authentication requests
- 📢 **General** - System alerts and general notifications

---

⭐ Star this repo if it helps secure your server!