# Day 09 - Troubleshoot and Restore MariaDB Service on Database Server

## Overview

This challenge focuses on troubleshooting a production database outage caused by a stopped MariaDB service. The application was unable to connect to the database, impacting normal operations.

Upon investigation, it was discovered that the MariaDB service could not start because the database had not been initialized and the required system tables were missing. The objective was to initialize the database, restore the service, and ensure it starts automatically after server reboots.

---

## Objective

- Investigate the MariaDB service failure
- Identify the root cause of the issue
- Initialize the MariaDB database files
- Restore the MariaDB service
- Configure the service to start automatically on boot
- Verify successful database operation

---

## Services Used

| Service | Purpose |
|----------|----------|
| MariaDB | Relational database service |
| systemd | Service management |
| Linux | System administration |
| mariadb-install-db | Initialize MariaDB database files |

---

## Implementation

### Check MariaDB Service Status

Verify the current status of the MariaDB service.

```bash
systemctl status mariadb
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `systemctl` | Controls and manages system services |
| `status` | Displays current service status |
| `mariadb` | Target MariaDB database service |

Expected Output:

```text
Active: inactive (dead)
```

or

```text
Database MariaDB is not initialized
```

This indicates that the service is not running.

---

### Attempt to Start MariaDB Service

Try starting the service to gather more information about the failure.

```bash
systemctl start mariadb
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `start` | Starts the specified service |
| `mariadb` | MariaDB database service |

Expected Error:

```text
Job for mariadb.service failed because the control process exited with error code.
```

---

### Recheck Service Status

Review the service status again to identify the root cause.

```bash
systemctl status mariadb
```

Expected Output:

```text
Database MariaDB is not initialized
```

#### Root Cause

MariaDB requires database system tables and files inside:

```text
/var/lib/mysql
```

Since these files were missing, MariaDB could not start successfully.

---

### Initialize the MariaDB Database

Create the required MariaDB system database files.

```bash
mariadb-install-db --user=mysql --ldata=/var/lib/mysql
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `mariadb-install-db` | Initializes MariaDB system tables |
| `--user=mysql` | Assigns ownership to the mysql user |
| `--ldata=/var/lib/mysql` | Specifies the database data directory |

Expected Output:

```text
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...
OK
```

This step creates the necessary database structure required for MariaDB operation.

---

### Start MariaDB Service

After initialization, start the MariaDB service.

```bash
systemctl start mariadb
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `start` | Starts the service immediately |
| `mariadb` | MariaDB database service |

---

## Verification

### Verify MariaDB Service Status

Confirm that the service is running successfully.

```bash
systemctl status mariadb
```

Expected Output:

```text
Active: active (running)
```

This confirms that MariaDB is operational.

---

### Enable MariaDB Service at Boot

Configure MariaDB to start automatically after system reboots.

```bash
systemctl enable mariadb
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `enable` | Configures automatic startup at boot |
| `mariadb` | MariaDB database service |

---

### Verify Boot-Time Configuration

Confirm that automatic startup has been configured.

```bash
systemctl is-enabled mariadb
```

Expected Output:

```text
enabled
```

This confirms MariaDB will start automatically after future reboots.

---

## Workflow

```text
┌─────────────────────┐
│ Application Fails   │
│ Database Connection │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Check MariaDB       │
│ Service Status      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Service Fails       │
│ To Start            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Identify Root Cause │
│ Database Not        │
│ Initialized         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Initialize MariaDB  │
│ Database Files      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Start MariaDB       │
│ Service             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Enable Auto Start   │
│ At Boot             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Database Service    │
│ Running Successfully│
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `systemctl status mariadb` | Check MariaDB service status |
| `systemctl start mariadb` | Start MariaDB service |
| `mariadb-install-db --user=mysql --ldata=/var/lib/mysql` | Initialize MariaDB database |
| `systemctl enable mariadb` | Enable service at boot |
| `systemctl is-enabled mariadb` | Verify boot-time startup configuration |

---

## Key Learnings

- Learned how to troubleshoot failed Linux services.
- Diagnosed MariaDB startup failures using systemd.
- Understood the importance of database initialization.
- Initialized MariaDB system tables manually.
- Configured service persistence across server reboots.

---

## Best Practices

- Always check service status before troubleshooting.
- Review service logs when startup failures occur.
- Verify database initialization after fresh installations.
- Enable critical services to start automatically at boot.
- Regularly monitor database health in production environments.

---

## Challenge Status

✅ Successfully Completed
