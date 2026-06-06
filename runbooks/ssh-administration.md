# SSH Administration Runbook

## Ubuntu Linux Environment

### Purpose

This runbook provides procedures for administering, troubleshooting, securing, and recovering Secure Shell (SSH) access on Ubuntu Linux servers.

### Scope

Applies to:

- Ubuntu Server 20.04 LTS
- Ubuntu Server 22.04 LTS
- Ubuntu Server 24.04 LTS

Services Covered:

- OpenSSH Server (sshd)
- SSH User Authentication
- SSH Key Management
- SSH Access Control
- SSH Incident Response

---

# 1. Verify SSH Service Status

## Check if SSH is installed

```bash
dpkg -l | grep ssh-server
```

## Check Service Status

```bash
sudo systemctl status ssh
```

Expected Output:

```text
Active: active (running)
```

## Check if SSH is Enabled at Boot

```bash
sudo systemctl is-enabled ssh
```

Expected Output:

```text
enabled
```

## Start SSH Service

```bash
sudo systemctl start ssh
```

## Stop SSH Service

```bash
sudo systemctl stop ssh
```

## Restart SSH Service

```bash
sudo systemctl restart ssh
```

## Reload Configuration Without Disconnecting Sessions

```bash
sudo systemctl reload ssh
```

---

# 2. Verify SSH Connectivity

## Check Listening Port

```bash
sudo ss -tulpn | grep ssh
```

Expected Output:

```text
LISTEN 0 128 0.0.0.0:22
```

## Verify Local Connectivity

```bash
ssh localhost
```

## Verify Remote Connectivity

```bash
ssh username@server-ip
```

---

# 3. SSH Configuration Management

Configuration File:

```bash
/etc/ssh/sshd_config
```

## Backup Configuration

```bash
sudo cp /etc/ssh/sshd_config \
/etc/ssh/sshd_config.backup
```

## Validate Configuration

Before restarting SSH:

```bash
sudo sshd -t
```

Expected Output:

```text
(no output)
```

---

# 4. User Access Management

## Create New User

```bash
sudo adduser username
```

## Add User to Sudo Group

```bash
sudo usermod -aG sudo username
```

## Lock User Account

```bash
sudo passwd -l username
```

## Unlock User Account

```bash
sudo passwd -u username
```

## Verify Account Status

```bash
sudo passwd -S username
```

---

# 5. SSH Key Management

## Generate Key Pair (Client)

```bash
ssh-keygen -t ed25519
```

## Copy Public Key to Server

```bash
ssh-copy-id username@server-ip
```

## Verify Authorized Keys

```bash
cat ~/.ssh/authorized_keys
```

## Correct Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

# 6. Security Hardening

## Disable Root Login

Edit:

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```text
PermitRootLogin no
```

## Disable Password Authentication

```text
PasswordAuthentication no
```

## Allow Only Specific Users

```text
AllowUsers adminuser cloudadmin
```

## Change Default Port

```text
Port 2222
```

Validate:

```bash
sudo sshd -t
```

Restart:

```bash
sudo systemctl restart ssh
```

---

# 7. Firewall Verification

## Check UFW Status

```bash
sudo ufw status
```

## Allow SSH

```bash
sudo ufw allow 22/tcp
```

## Allow Alternate SSH Port

```bash
sudo ufw allow 2222/tcp
```

## Reload Firewall

```bash
sudo ufw reload
```

---

# 8. Log Monitoring

## View SSH Logs

Ubuntu 22.04+

```bash
sudo journalctl -u ssh
```

Real-Time Monitoring:

```bash
sudo journalctl -fu ssh
```

Authentication Events:

```bash
sudo grep sshd /var/log/auth.log
```

Failed Login Attempts:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Successful Logins:

```bash
sudo grep "Accepted" /var/log/auth.log
```

---

# 9. Incident Response Procedures

## Incident A: Connection Refused

### Symptoms

```text
ssh: connect to host x.x.x.x port 22:
Connection refused
```

### Investigation

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep ssh
```

### Resolution

```bash
sudo systemctl start ssh
```

---

## Incident B: Connection Timed Out

### Symptoms

```text
Operation timed out
```

### Investigation

```bash
sudo ufw status
ping server-ip
```

### Resolution

Verify:

- Network connectivity
- Firewall rules
- Security groups (AWS/Azure/GCP)

---

## Incident C: Permission Denied

### Symptoms

```text
Permission denied (publickey)
```

### Investigation

```bash
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

### Resolution

Correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Reinstall key if necessary.

---

## Incident D: SSH Service Fails to Start

### Investigation

```bash
sudo systemctl status ssh
sudo journalctl -xe
sudo sshd -t
```

### Common Root Cause

```text
Invalid sshd_config directive
```

### Resolution

Restore backup:

```bash
sudo cp \
/etc/ssh/sshd_config.backup \
/etc/ssh/sshd_config
```

Restart:

```bash
sudo systemctl restart ssh
```

---

# 10. Recovery Checklist

Verify:

- SSH service running
- Port listening
- Firewall rule present
- Network connectivity operational
- User account unlocked
- SSH key valid
- Configuration validated
- Logs reviewed

Validation Commands:

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep ssh
sudo ufw status
sudo sshd -t
```

---

# Escalation Criteria

Escalate if:

- Root filesystem full
- Suspected compromise
- Repeated brute-force attacks
- Service repeatedly crashes
- Authentication subsystem corruption
- Multiple production servers affected

---

# References

OpenSSH Documentation

```bash
man ssh
man sshd
man sshd_config
```

Ubuntu Service Management

```bash
man systemctl
```

To check if a service is already instaled on your linux machine:

1. List all installed packages and filter output bu specific package name

```
dpkg -l | grep <service-name>
```

For example, to check if open ssh is installed:

```
dpkg -l | grep openssh-client
```

2. Check version

'''
<service-name> -V
'''
