# 🌺 𝓜𝓲𝓵𝓵𝓱𝓲𝓸𝓻𝓮 𝓣𝓕𝓢 𝓓𝓸𝔀𝓷𝓰𝓻𝓪𝓭𝓮 🌺
### 🐱 [Based nekiro downgrade.](https://github.com/nekiro/TFS-1.5-Downgrades)

- This downgrade is not download and run distribution, monsters and spells are probably not 100% correct.
- You are welcome to submit a pull request though.

## 🛠 It is currently under development. ⚙

## 📋 Table of Contents
- [Compilation](#compilation)
  - [Ubuntu 22.04](#compiling-on-ubuntu-2204)
  - [Windows](#compiling-on-windows)
- [Database Setup](#database-setup)
- [Server Configuration](#server-configuration)
- [Firewall & Ports](#firewall--ports)
- [Troubleshooting](#troubleshooting)

---

## Compilation

### Compiling on Ubuntu 22.04

#### 1. Install Dependencies
```bash
sudo apt update
sudo apt install -y git cmake build-essential libluajit-5.1-dev libmysqlclient-dev \
    libboost-system-dev libboost-date-time-dev libboost-filesystem-dev \
    libboost-iostreams-dev libboost-locale-dev libcrypto++-dev \
    libpugixml-dev libfmt-dev libssl-dev zlib1g-dev
```

#### 2. Clone Repository
```bash
git clone https://github.com/MillhioreBT/forgottenserver-downgrade.git
cd forgottenserver-downgrade
```

#### 3. Compile (Choose one)

**Debug Build:**
```bash
mkdir -p build-debug
cd build-debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build . -- -j$(nproc)
```

**Release Build (Recommended for production):**
```bash
mkdir -p build-release
cd build-release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . -- -j$(nproc)
```

The executable will be at `build-debug/tfs` or `build-release/tfs`

### Compiling on Windows

[See Windows Compilation Guide](https://github.com/MillhioreBT/forgottenserver-downgrade/wiki/Compiling-on-Windows-(vcpkg))

---

## Database Setup

### 1. Install MySQL/MariaDB
```bash
sudo apt install mysql-server -y
```

### 2. Create Database and User
```bash
sudo mysql -u root -p
```

Inside MySQL:
```sql
CREATE DATABASE forgottenserver CHARACTER SET utf8;
CREATE USER 'forgottenserver'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON forgottenserver.* TO 'forgottenserver'@'localhost';
FLUSH PRIVILEGES;
USE forgottenserver;
SOURCE /path/to/schema.sql;
EXIT;
```

### 3. Verify Database
```bash
mysql -u forgottenserver -p forgottenserver -e "SHOW TABLES; SELECT id, name FROM accounts;"
```

**Default Account:**
- Username: `1`
- Password: `1`

---

## Server Configuration

### 1. Create config.lua
```bash
cp config.lua.dist config.lua
nano config.lua
```

### 2. Essential Configuration

**Database Settings (around line 100):**
```lua
mysqlHost = "127.0.0.1"
mysqlUser = "forgottenserver"
mysqlPass = "your_password"
mysqlDatabase = "forgottenserver"
mysqlPort = 3306
```

**Network Settings (around line 44):**
```lua
ip = "YOUR_PUBLIC_IP"  -- Use your VPS public IP, NOT 127.0.0.1!
bindOnlyGlobalAddress = false
loginProtocolPort = 7171
gameProtocolPort = 7172
statusProtocolPort = 7171
```

**Get your public IP:**
```bash
curl ifconfig.me
```

### 3. Account Manager (Optional)
```lua
accountManager = true  -- Allows login with empty user/pass to create accounts
```

---

## Firewall & Ports

### Ubuntu/Debian (UFW)
```bash
sudo ufw allow 7171/tcp  # Login port
sudo ufw allow 7172/tcp  # Game port
sudo ufw status
```

### Oracle Cloud (Additional Steps Required)

If using Oracle Cloud VPS, also configure in the web console:

1. Go to **Networking → Virtual Cloud Networks**
2. Select your VCN
3. Click **Security Lists** → Default Security List
4. Add **Ingress Rules**:
   - **Source CIDR:** `0.0.0.0/0`
   - **IP Protocol:** TCP
   - **Destination Port:** `7171`
   - Repeat for port `7172`

### Verify Ports are Open
```bash
sudo ss -tlnp | grep -E '7171|7172'
```

Should show:
```
LISTEN 0  4096  0.0.0.0:7171  0.0.0.0:*  users:(("tfs",pid=XXXXX,fd=XX))
LISTEN 0  4096  0.0.0.0:7172  0.0.0.0:*  users:(("tfs",pid=XXXXX,fd=XX))
```

---

## Troubleshooting

### ❌ "Cannot connect to game server" / "Invalid server address (10049)"

**Solution:** TFS is listening on 127.0.0.1 instead of public IP

```bash
# Check current IP configuration
grep "^ip = " config.lua

# Should show your public IP, not 127.0.0.1
# If wrong, edit config.lua and set: ip = "YOUR_PUBLIC_IP"

# Restart server
sudo pkill -9 tfs
./tfs
```

### ❌ "Account name or password is not correct"

**Solutions:**

1. **Use Account Manager:**
   - Set `accountManager = true` in config.lua
   - Leave username and password **EMPTY** in client

2. **Create a character:**
   ```sql
   USE forgottenserver;
   INSERT INTO players (name, account_id, level, vocation, health, healthmax, 
                        looktype, town_id, posx, posy, posz, cap, sex)
   VALUES ('YourCharacter', 1, 8, 0, 185, 185, 128, 1, 95, 117, 7, 470, 1);
   ```

3. **Verify credentials:**
   ```bash
   mysql -u forgottenserver -p -e "SELECT id, name, password FROM accounts;" forgottenserver
   ```

### ❌ MySQL Connection Failed

```bash
# Test MySQL connection
mysql -u forgottenserver -p -h 127.0.0.1 forgottenserver

# Check if MySQL is running
sudo systemctl status mysql

# Check TFS logs when starting
./tfs
# Should see: ">> MySQL connection successful!"
```

### 🔍 Verify Server Status

```bash
# Check if TFS is running
ps aux | grep tfs

# Check listening ports
sudo ss -tlnp | grep tfs

# Check MySQL connections
sudo mysql -u root -p -e "SHOW PROCESSLIST;"

# View active firewall rules
sudo ufw status numbered
```

### 🔄 Clean Restart

```bash
# Kill all TFS processes
sudo pkill -9 tfs

# Verify nothing is running
ps aux | grep tfs

# Start fresh
cd ~/forgottenserver-downgrade/build-release
./tfs
```

---

## Contributing

Pull requests are welcome.
Just make sure you are using english/spanish language.

## Bugs

If you find any bug and believe it should be fixed, submit an issue in [issues section](https://github.com/MillhioreBT/forgottenserver-downgrade/issues), just please follow the issue template.

---

## 📚 Additional Resources

- [Official Wiki](https://github.com/MillhioreBT/forgottenserver-downgrade/wiki)
- [Original TFS Documentation](https://github.com/otland/forgottenserver)
- [OTServ Brasil Forum](https://forums.otserv.com.br/)
