# SonarQube Upgrade Automation using Ansible

This project automates the **SonarQube upgrade process** using Ansible.  
It performs service management, backup, extraction, configuration updates, and permission corrections to ensure a smooth upgrade.

---

## 📁 Project Structure

```
sonar-upgrade-automation/
└── automation-sonar/
    └── tasks/
        ├── main.yml
        ├── stop-service.yml
        ├── start-service.yml
        ├── tarback.yml
        ├── backup.yml
        ├── extract.yml
        ├── rename2.yml
        ├── system-service2.yml
        ├── append-sonar.yml
        └── permission.yml
```

---

## 🔄 Upgrade Workflow

### Step 1 – Stop Services
Stops both **PostgreSQL** and **SonarQube** services.  
File: `stop-service.yml`

---

### Step 2 – Backup Existing SonarQube
Creates a backup of the current SonarQube working directory:

```
/opt/sonarqube/
```

The archive is stored as:

```
/tmp/sonarqube-{{ current_date }}.tar
```

File: `tarback.yml`

---

### Step 3 – Download SonarQube Package (Manual Step)

Download the latest SonarQube `.zip` file from the official website.

- **Developer Edition** → Requires authentication (not publicly accessible)
- **Community Edition** → Publicly available

⚠ This step is manual for Developer Edition due to restricted access.

---

### Step 4 – Copy & Extract New Version

Copy the downloaded file:

```
/tmp/{{ latest_version.zip }}
```

To:

```
/opt/sonarqube/
```

Then extract the archive.

File: `extract.yml`

---

### Step 5 – Rename Extracted Directory

Rename the extracted folder using the format:

```
SonarQube-MAJOR.MINOR.PATCH
```

Example:

```
SonarQube-9.9.1
```

File: `rename2.yml`

---

### Step 6 – Update System Service File

Replace the old SonarQube version path with the new version in the system service configuration.

File: `system-service2.yml`

---

### Step 7 – Update sonar.properties

Update `sonar.properties` with:

- JDBC connection string  
- Database credentials  

File: `append-sonar.yml`

---

### Step 8 – Fix Permissions & Ownership

Ensure proper ownership and permissions for SonarQube directories and files.

File: `permission.yml`

---

### Step 9 – Start Services

Start both:

- PostgreSQL  
- SonarQube  

File: `start-service.yml`

---

## 🚀 How to Run

From the root directory:

```bash
ansible-playbook main.yml
```

---

## ✅ Features

- Automated backup before upgrade
- Safe service stop/start handling
- Configurable database settings
- Structured version naming
- Reusable Ansible task-based design

---

## 📌 Notes

- Verify database compatibility before upgrading SonarQube.
- Ensure sufficient disk space before backup and extraction.
- Test upgrades in a staging environment before applying to production.
