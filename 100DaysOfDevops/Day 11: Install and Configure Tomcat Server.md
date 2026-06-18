# Day 11 - Deploy a Java Application on Apache Tomcat

## Overview

The Nautilus application development team has completed the beta version of a Java-based web application and decided to deploy it using Apache Tomcat on App Server 1. The application is packaged as a `ROOT.war` file and must be accessible directly from the base URL without any additional context path.

---

## Objective

- Install Apache Tomcat on App Server 1
- Configure Tomcat to listen on port `8084`
- Deploy the provided `ROOT.war` application
- Ensure the application is accessible from the base URL
- Verify successful deployment

---

## Services Used

| Service | Purpose |
|----------|----------|
| Apache Tomcat | Java Application Server |
| SSH | Remote server access |
| SCP | Secure file transfer |
| Systemd | Service management |
| Linux | Server administration |

---

## Implementation

### Connect to App Server 1

Login to App Server 1 from the Jump Host.

```bash
ssh tony@stapp01
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `ssh` | Securely connects to a remote server |
| `tony` | User account on App Server 1 |
| `stapp01` | Hostname of App Server 1 |

---

### Switch to Root User

Administrative privileges are required for package installation and configuration.

```bash
sudo su -
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `sudo` | Execute commands with elevated privileges |
| `su -` | Switch to root user with root environment |

---

### Install Tomcat Packages

Install Tomcat and required web application packages.

```bash
yum install -y tomcat tomcat-webapps tomcat-admin-webapps
```

#### Command Explanation

| Parameter | Description |
|------------|-------------|
| `yum install` | Installs software packages |
| `-y` | Automatically accepts prompts |
| `tomcat` | Core Tomcat application server |
| `tomcat-webapps` | Default Tomcat web applications |
| `tomcat-admin-webapps` | Administrative web applications |

---

### Configure Tomcat Port

Open the Tomcat configuration file.

```bash
vi /etc/tomcat/server.xml
```

Find the following line:

```xml
Connector port="8080"
```

Replace it with:

```xml
Connector port="8084"
```

Save and exit:

```text
ESC
:wq
```

#### Configuration Explanation

| Setting | Description |
|----------|-------------|
| `server.xml` | Main Tomcat configuration file |
| `Connector port` | Defines the HTTP listening port |
| `8084` | Required port for this deployment |

---

### Copy Application from Jump Host

From the Jump Host, transfer the application package to App Server 1.

```bash
scp /tmp/ROOT.war tony@stapp01:/tmp/
```

#### Command Explanation

| Component | Description |
|------------|-------------|
| `scp` | Secure copy utility |
| `/tmp/ROOT.war` | Source application package |
| `tony@stapp01:/tmp/` | Destination location on App Server 1 |

---

### Remove Default ROOT Application

Remove the default Tomcat ROOT application to avoid deployment conflicts.

```bash
rm -rf /usr/share/tomcat/webapps/ROOT
rm -f /usr/share/tomcat/webapps/ROOT.war
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `rm -rf` | Removes directory and contents recursively |
| `rm -f` | Removes file without confirmation |
| `ROOT` | Default Tomcat application |

---

### Deploy the New Application

Copy the provided application package into the Tomcat deployment directory.

```bash
cp /tmp/ROOT.war /usr/share/tomcat/webapps/
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `cp` | Copies files |
| `ROOT.war` | Application archive |
| `webapps` | Tomcat auto-deployment directory |

---

### Enable and Start Tomcat

Enable the service to start automatically on boot.

```bash
systemctl enable tomcat
```

Restart Tomcat to apply configuration changes and deploy the application.

```bash
systemctl restart tomcat
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `systemctl enable tomcat` | Starts Tomcat automatically on boot |
| `systemctl restart tomcat` | Reloads configuration and redeploys applications |

---

## Verification

### Verify Tomcat Service Status

```bash
systemctl status tomcat
```

Expected Output:

```text
Active: active (running)
```

---

### Verify Application Deployment

Access the application using the base URL.

```bash
curl http://stapp01:8084
```

Expected Output:

```html
<h2>Welcome to xFusionCorp Industries!</h2>
```

This confirms:

- Tomcat is running successfully
- Port 8084 is configured correctly
- ROOT.war has been deployed successfully
- Application is accessible from the base URL

---

### Optional Port Verification

Using `ss`:

```bash
ss -tlnp | grep 8084
```

Or using `netstat`:

```bash
netstat -tlnp | grep 8084
```

Expected Output:

```text
LISTEN 0 128 :::8084 :::*
```

---

## Deployment Workflow

```text
┌─────────────────────┐
│ Login to stapp01    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Install Tomcat      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Change Port 8080    │
│ to 8084             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Copy ROOT.war       │
│ from Jump Host      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Remove Default      │
│ ROOT Application    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Deploy ROOT.war     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Restart Tomcat      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify via curl     │
│ http://stapp01:8084 │
└─────────────────────┘
```

---

## Command Reference

| Command | Purpose |
|----------|----------|
| `ssh tony@stapp01` | Connect to App Server 1 |
| `sudo su -` | Switch to root user |
| `yum install -y tomcat` | Install Tomcat packages |
| `vi /etc/tomcat/server.xml` | Edit Tomcat configuration |
| `scp /tmp/ROOT.war ...` | Transfer application package |
| `cp ROOT.war webapps/` | Deploy application |
| `systemctl restart tomcat` | Restart Tomcat service |
| `systemctl status tomcat` | Verify service status |
| `curl http://stapp01:8084` | Verify application access |

---

## Key Learnings

- Learned how to install Apache Tomcat on Linux.
- Understood Tomcat port configuration through `server.xml`.
- Practiced deploying Java applications using WAR files.
- Learned how ROOT applications work in Tomcat.
- Verified deployments using HTTP requests and service checks.

---

## Best Practices

- Always backup configuration files before modification.
- Remove unused default applications to reduce attack surface.
- Verify application deployment after every restart.
- Use custom ports when required by organizational standards.
- Enable services to ensure persistence after reboot.

---

## Challenge Status

✅ Successfully Completed
