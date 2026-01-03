🔐 SSH Hardening & Security – Part 1


📌 Objective

Implement security hardening measures on an SSH server to reduce unauthorized access risks and prepare the environment for future SOC (Security Operations Center) integrations.

⚙️ Environment

Operating System: Arch Linux (Virtual Machine on Oracle VirtualBox)

Tools used:

OpenSSH (sshd)

iptables (firewall)

systemd / systemctl (service management)


🛡️ Applied Configurations

1️⃣ Default SSH Port Change

The default SSH port was changed to reduce exposure to automated scans and brute-force attempts.

Configuration file:

/etc/ssh/sshd_config

Change applied:

Port 2222

Result:

Port 22 filtered/blocked
SSH service accessible only via port 2222
Note: This measure does not replace proper authentication controls but helps reduce noise from automated attacks.

2️⃣ Public Key Authentication

Password-based authentication was disabled in favor of SSH key-based authentication.
Key generation (client-side):

ssh-keygen -t ed25519

Key deployment to server:

ssh-copy-id -p 2222 m4@server

sshd configuration:

PasswordAuthentication no

Result:

Password login disabled
Access allowed only via cryptographic keys

3️⃣ Firewall Configuration (iptables)

Firewall rules were implemented to explicitly allow SSH access only on the hardened port.

Rules applied:

sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 22 -j DROP

Persistence:

sudo iptables-save > /etc/iptables/iptables.rules

sudo systemctl enable iptables

Result:
Port 22 fully blocked

Only port 2222 allowed for SSH connections

4️⃣ SSH Service Management

Service control was tested to validate hardening and containment scenarios.

Disable SSH service:

sudo systemctl stop sshd

sudo systemctl disable sshd

sudo systemctl mask sshd

Re-enable SSH service:

sudo systemctl unmask sshd

sudo systemctl enable sshd

sudo systemctl start sshd

Purpose:

Validate service exposure control

Simulate service shutdown and recovery scenarios

📊 Tests Performed
🔍 Nmap Scan

Before: Port 22 open
After: Port 22 filtered, port 2222 open

🔐 SSH Connection Test

ssh -p 2222 m4@server

Result:

Successful login using SSH key authentication only
Password authentication rejected
✅ Outcome
This hardening process significantly reduces the SSH attack surface and establishes a secure baseline for future SOC-oriented monitoring and network defense projects.