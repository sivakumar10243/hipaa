# Ubuntu - MariaDB File Key Management Setup Guide
---
## Step 1: Create Key Directory

```bash
sudo mkdir -p /etc/mysql/encryption
sudo chown mysql:mysql /etc/mysql/encryption
sudo chmod 750 /etc/mysql/encryption
```

---

## Step 2: Generate Encryption Keys (Automated Recommended)

```bash
for i in 1 2 3; do
    echo "$i;$(openssl rand -hex 32)" | sudo tee -a /etc/mysql/encryption/keyfile.txt
done
```

**Secure the key file:**

```bash
sudo chown mysql:mysql /etc/mysql/encryption/keyfile.txt
sudo chmod 600 /etc/mysql/encryption/keyfile.txt
```

---

## Step 3: Configure MariaDB

Edit `/etc/mysql/mariadb.conf.d/50-server.cnf` and add under `[mysqld]`:

```ini
# --- Encryption ---
plugin_load_add = file_key_management
file_key_management_filename = /etc/mysql/encryption/keyfile.txt
file_key_management_encryption_algorithm = AES_CTR
innodb_encrypt_tables = ON
innodb_encrypt_temporary_tables = ON
innodb_encrypt_log = ON
innodb_encryption_threads = 4
innodb_encryption_rotate_key_age = 1

# Performance settings
innodb_buffer_pool_size = 128M
innodb_max_dirty_pages_pct = 5
innodb_max_dirty_pages_pct_lwm = 1
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT
innodb_adaptive_flushing = OFF
innodb_flushing_avg_loops = 1
```

Restart MariaDB:

```bash
sudo systemctl restart mariadb
```

---

## **Step 4: Verify Encryption**

Inside the MariaDB prompt:

```sql
-- Check if the encryption plugin is active
SHOW PLUGINS;
```

**Expected output includes:**
```
+-----------------------+--------+------------+----------------------------+-------+
| Name                  | Status | Type       | Library                    | License |
+-----------------------+--------+------------+----------------------------+-------+
| file_key_management   | ACTIVE | ENCRYPTION | file_key_management.dll    | GPL    |
+-----------------------+--------+------------+----------------------------+-------+
```
---