# Secure File Storage and Access Management for Project Teams

## 📝 Project Overview
This project demonstrates the implementation of enterprise-grade access control, session-hardening policies, and security auditing mechanisms on a Linux (Kali/Ubuntu) platform. The architecture segregates distinct project workgroups (`proja` and `projb`), configures granular user permissions via Linux Access Control Lists (ACLs), implements security restrictions using the Sticky Bit, enforces corporate command logging parameters, and deploys an automated web-based reporting system utilizing Apache2 and `auditd`.

---

## 🎯 Objectives
* **Implement Role-Based Access Control (RBAC):** Restrict shared folder access strictly to authorized project group members.
* **Enforce Granular Permissions:** Leverage advanced POSIX ACLs and Sticky Bit modifications to restrict cross-user file deletions within shared spaces.
* **Harden Terminal Sessions:** Restrict user command history visibility (`HISTSIZE`) based on organizational access tiers.
* **Automate Security Monitoring:** Configure the Linux Audit Daemon (`auditd`) to watch for directory modifications and automate a restricted, web-accessible compliance log portal.

---

## 🛠️ Tools & Technologies Used
* **Operating System:** Kali Linux / Bash Environment
* **Security Auditing Engine:** Audit Daemon (`auditd`, `auditctl`, `ausearch`)
* **Web Server Reporting:** Apache2 (Secured via `.htaccess` basic authentication)
* **Task Automation:** Crontab
* **Access Control Mechanisms:** Linux Groups, standard chmod/chown, POSIX Access Control Lists (ACLs), Sticky Bit (`+t`)

---

## 🚀 Step-by-Step Implementation

### Phase 1: Identity and Access Management Configuration
1. **Provision Groups and Users:** Created isolated developer pools for Project A (`proja`) and Project B (`projb`), adding multi-tier user endpoints to simulate real workplace separation.
   ```bash
   sudo groupadd proja
   sudo groupadd projb
   sudo useradd -m -g proja pa1
   sudo useradd -m -g projb pb1
   sudo passwd pa1
   ```
2. **Shared Storage Sandbox Initialization:** Provisions the root project repository and explicitly aligns permissions so that unauthorized OS users are fundamentally excluded.
   ```bash
   sudo mkdir /home/project
   sudo chown :projb /home/project
   sudo chmod 770 /home/project
   ```

### Phase 2: Advanced Permissions and Hardening
1. **POSIX Access Control Lists (ACL) Definition:** Configured explicit read-write-execute rule sets per user and managed folder fallback definitions to guarantee files inherit parent protections automatically.
   ```bash
   sudo setfacl -m u:pa1:rwx /home/project
   sudo setfacl -m g::r-x /home/project
   sudo setfacl -d -m u::rwx /home/project
   sudo setfacl -d -m o::--- /home/project
   ```
2. **Sticky Bit Prevention:** Deployed the terminal Sticky bit (`+t`) to prevent workspace manipulation, guaranteeing users cannot alter or destroy work belonging to peer teammates within the shared root.
   ```bash
   sudo chmod +t /home/project
   ```
3. **Session Compliance Controls:** Aligned default profile baselines to use standard `/bin/bash` and restricted terminal recall indexes (`HISTSIZE`) to maintain corporate data boundaries.
   ```bash
   sudo chsh -s /bin/bash pa1
   echo "HISTSIZE=10" | sudo tee -a /home/pa1/.bashrc
   ```

### Phase 3: SIEM Auditing & Reporting Automation
1. **Auditing Ruleset Ingestion:** Initialized the internal system monitor daemon (`auditd`) and appended structured system trace definitions onto file pointer targets.
   ```bash
   sudo apt-get install auditd -y
   sudo auditctl -w /home/project -p rwxa -k project_access
   ```
2. **Persistent Rule Preservation:** Appended parameters manually inside `/etc/audit/rules.d/audit.rules` to enforce continuous validation across system power cycles:
   ```text
   -w /home/project -p rwxa -k project_access
   ```
3. **Automated SIEM Dashboard Pipelines:** Provisioned an automated task manager thread running every five minutes to compile trace collections cleanly into public server roots.
   ```bash
   # Configured via crontab -e
   */5 * * * * ausearch -k project_access >> /var/www/html/auditlog.txt
   ```
4. **Apache Portal Security:** Configured endpoint security maps inside `/etc/apache2/sites-available/000-default.conf` using an encrypted `.htaccess` user table file path to safeguard network monitoring information from external view.
   ```text
   AuthType Basic
   AuthName "Restricted Access"
   AuthUserFile /var/www/html/.htpasswd
   Require valid-user
   ```

---

## 🧪 Verification & Defensive Validation

### 1. Cross-Deletion Validation (Sticky Bit Test)
* **Action:** Logged in as user `pa1`, populated `testfile.txt`. Switched active sessions to alternative context user `pa2` and ran destructive arguments.
* **Outcome:** System successfully drop-rejected commands with an explicit `Operation not permitted` warning, indicating the folder environment blocked file deletions by anyone other than the data owner.

### 2. Boundary Escalation Validation (Unauthorized User Test)
* **Action:** Logged in as local context testing entity `test` and attempted explicit workspace traversal.
* **Outcome:** Traversal dropped cleanly with a `-sh: cd: can't cd to /home/project` operational rejection.

### 3. Log Exfiltration Validation (Secure Web Report Test)
* **Action:** Routed client web engine requests through endpoint tracking definitions (`http://localhost/auditlog.txt`).
* **Outcome:** Browser session systematically halted on an authorization challenge before cleanly resolving verified `auditd` logging sequences securely over low-profile HTTP frameworks.

---

## 🧠 Key Takeaways
* **Granular Rule Dominance:** Traditional Unix permission bounds (`rwxrwxrwx`) prove too rigid for enterprise workforce settings. Deploying targeted POSIX Access Control Lists (ACLs) permits precise security policies without shifting primary system owners.
* **The Importance of Sticky Bits:** Even in folders where group collaboration requires complete read/write access, adding the Sticky Bit (`+t`) is essential to prevent internal data loss or accidental deletion incidents.
* **System Event Security:** Monitoring events using toolchains like `auditd` provides reliable log collection, but these files must always be moved out of default directories into protected, access-controlled tracking portals to ensure secure infrastructure auditing.
