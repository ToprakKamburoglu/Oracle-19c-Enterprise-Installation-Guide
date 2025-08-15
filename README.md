
# Oracle Database 19c Enterprise Edition Installation Guide

<div align="center">
  <img src="images/Figes.png" alt="Figes Logo" width="500"/>
</div>

<div align="center">

[![Oracle](https://img.shields.io/badge/Oracle-Database%2019c-red?style=flat-square&logo=oracle)](https://www.oracle.com/database/)
[![Linux](https://img.shields.io/badge/Oracle%20Linux-8.10-orange?style=flat-square&logo=linux)](https://www.oracle.com/linux/)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-Virtualization-blue?style=flat-square&logo=virtualbox)](https://www.virtualbox.org/)
[![Enterprise](https://img.shields.io/badge/Edition-Enterprise-gold?style=flat-square)](https://www.oracle.com/database/enterprise-edition/)

</div>

This project provides a comprehensive guide for installing and configuring Oracle Database 19c (19.3) Enterprise Edition on Oracle Linux 8.10 using Oracle VirtualBox virtualization platform. You can check the detailed guide of "IFS_Oracle 19c Installation and Configuration – Oracle Linux 8.10.pdf". 

## 📋 Table of Contents

| Section              | Link |
|----------------------|------|
| Overview             | [Go to Section](#-overview) |
| System Requirements  | [Go to Section](#-system-requirements) |
| Installation Steps   | [Go to Section](#-installation-steps) |
| Configuration        | [Go to Section](#-configuration) |
| Security Settings    | [Go to Section](#-security-settings) |
| Backup Procedures    | [Go to Section](#-backup-procedures) |
| Troubleshooting      | [Go to Section](#-troubleshooting) |
| Contributing         | [Go to Section](#-contributing) |

## 🎯 Overview

This guide covers the complete installation and configuration process including:

### Project Details
- **Virtualization Platform**: Oracle VirtualBox
- **Operating System**: Oracle Linux 8.10 UEK x86-64
- **Database Version**: Oracle Database 19c (19.3) Enterprise Edition
- **Container Database (CDB)**: TESTCDB
- **Pluggable Database (PDB)**: TESTPDB

### 🏛️ Architecture
```
┌─────────────────────────────────────────┐
│           Host Operating System          │
├─────────────────────────────────────────┤
│            Oracle VirtualBox            │
├─────────────────────────────────────────┤
│          Oracle Linux 8.10             │
├─────────────────────────────────────────┤
│     Oracle Database 19c Enterprise      │
│    ┌─────────────┬─────────────────┐    │
│    │   TESTCDB   │    TESTPDB      │    │
│    │    (CDB)    │     (PDB)       │    │
│    └─────────────┴─────────────────┘    │
└─────────────────────────────────────────┘
```

## 💻 System Requirements

### Hardware Requirements
- **vCPU**: 2-4 cores
- **RAM**: 6-8 GB (minimum)
- **Disk Space**: 50 GiB virtual disk
- **Host RAM**: Minimum 8 GB recommended

### Software Requirements
- Oracle VirtualBox (Latest version)
- Oracle VirtualBox Extension Pack
- Oracle Linux 8.10 UEK x86-64 Full ISO
- Oracle Database 19c (19.3) Enterprise Edition

### Network Requirements
- Internet connection for downloads
- NAT or Bridged network configuration

## 🚀 Installation Steps

### Task 1: Oracle VirtualBox Setup

#### 1.1 VirtualBox Installation
1. **Download VirtualBox**
   ```bash
   # Download from Oracle VirtualBox official website
   # Choose appropriate version for Windows/macOS/Linux
   ```

2. **Install Extension Pack**
   - Download `Oracle_VirtualBox_Extension_Pack-7.1.6.vbox-extpack`
   - VirtualBox Manager → File → Tools → Extensions → Install Extension Pack

3. **Verify Installation**
   - Check if extension pack is properly installed
   - Verify VirtualBox version compatibility

### Task 2: Virtual Machine Creation

#### 2.1 VM Configuration
1. **Basic Settings**
   ```
   Name: Oracle Linux 8.10
   Type: Linux
   Subtype: Oracle Linux
   Version: Oracle Linux (64-bit)
   ```

2. **Hardware Configuration**
   ```
   Base Memory: 8192 MB (8 GB)
   Processors: 4 CPU
   Enable EFI: false
   ```

3. **Virtual Hard Disk**
   ```
   Type: VDI (VirtualBox Disk Image)
   Storage: Dynamically allocated
   Size: 50.00 GB
   Pre-allocate Full Size: false
   ```

#### 2.2 VM Properties Summary
```
Machine Name: Oracle Linux 8.10
Operating System: Oracle Linux (64-bit)
Base Memory: 8192 MB
Processors: 4
Boot Order: Floppy, Optical, Hard Disk
Acceleration: VT-x/AMD-V, Nested Paging
```

#### 2.3 ISO Configuration
1. **Attach Oracle Linux ISO**
   - Storage → Controller IDE → Optical Drive
   - Choose disk file: `OracleLinux-R8-U10-x86_64-dvd.iso`
   - Type: Image
   - Size: 13.18 GB

### Task 3: Oracle Database 19c Installation

#### 3.1 Prerequisites
1. **Network Connectivity Test**
   ```bash
   ping www.google.com
   ```

2. **Install Oracle Database Pre-installation Package**
   ```bash
   sudo dnf install -y oracle-database-preinstall-19c
   ```

3. **Verify Oracle User and DBA Groups**
   ```bash
   # Check Oracle user existence
   cat /etc/passwd | grep -i oracle
   
   # Check DBA groups
   grep -i dba /etc/group
   ```

#### 3.2 Oracle User Configuration
1. **Set Oracle User Password**
   ```bash
   sudo passwd oracle
   # Enter new password when prompted
   ```

2. **Create Directory Structure**
   ```bash
   sudo mkdir -p /u01/app/oracle/product/19.3/dbhome_1
   sudo mkdir -p /u02/oradata
   sudo chown -R oracle:oinstall /u01 /u02
   sudo chmod -R 775 /u01 /u02
   ```

#### 3.3 Environment Setup
1. **Create Environment Script**
   ```bash
   # Switch to oracle user
   su - oracle
   
   # Create scripts directory
   mkdir /home/oracle/scripts
   
   # Create setEnv.sh
   cat > /home/oracle/scripts/setEnv.sh <<EOF
   # Oracle Settings
   export TMP=/tmp
   export TMPDIR=\$TMP
   
   export ORACLE_HOSTNAME=mydb.localdomain
   export ORACLE_UNQNAME=TESTCDB
   export ORACLE_BASE=/u01/app/oracle
   export ORACLE_HOME=\$ORACLE_BASE/product/19.3/dbhome_1
   export ORA_INVENTORY=/u01/app/oraInventory
   export ORACLE_SID=TESTCDB
   export PDB_NAME=TESTPDB
   export DATA_DIR=/u02/oradata
   
   export PATH=/usr/sbin:/usr/local/bin:\$PATH
   export PATH=\$ORACLE_HOME/bin:\$PATH
   
   export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib
   export CLASSPATH=\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib
   EOF
   ```

2. **Update Bash Profile**
   ```bash
   echo ". /home/oracle/scripts/setEnv.sh" >> /home/oracle/.bash_profile
   ```

3. **Verify Environment Variables**
   ```bash
   source scripts/setEnv.sh
   echo $ORACLE_HOME
   cd $ORACLE_HOME
   ```

#### 3.4 Oracle Software Installation
1. **Prepare for Installation**
   ```bash
   cd /home/oracle
   chmod +x runInstaller
   export DISPLAY=:0
   ```

2. **Run Oracle Universal Installer**
   ```bash
   ./runInstaller
   ```

3. **Installation Configuration**
   - **Configuration Option**: Create and configure a single instance database
   - **System Class**: Server Class
   - **Database Installation Option**: Enterprise Edition
   - **Installation Location**: `/u01/app/oracle/product/19.3/dbhome_1`
   - **Configuration Type**: General Purpose/Transaction Processing
   - **Database Identifiers**: 
     - Global Database Name: TESTCDB
     - Oracle System Identifier (SID): TESTCDB
   - **Configuration Options**:
     - Memory: Automatic Shared Memory Management
     - SGA Size: 3072 MB
     - PGA Size: 1024 MB
   - **Management Options**: Default settings
   - **Operating System Groups**:
     - Database Administrator (OSDBA): dba
     - Database Operator (OSOPER): oper
     - Database Backup and Recovery (OSBACKUPDBA): backupdba
     - Data Guard administrative (OSDGDBA): dgdba
     - Encryption Key Management (OSKMDBA): kmdba
     - Real Application Cluster administrative (OSRACDBA): racdba

#### 3.5 Database Creation with DBCA
1. **Launch Database Configuration Assistant**
   ```bash
   su - oracle
   dbca
   ```

2. **DBCA Configuration Steps**
   - **Database Operation**: Create a database
   - **Creation Mode**: Advanced Configuration
   - **Deployment Type**: Oracle Single Instance database
   - **Database Type**: General Purpose or Transaction Processing
   - **Database Identification**:
     - Global Database Name: TESTCDB
     - SID: TESTCDB
     - Create as Container database: ✓
     - Use Local Undo tablespace for PDBs: ✓
     - Number of PDBs: 1
     - PDB Name: TESTPDB

3. **Memory Configuration**
   ```
   Use Automatic Shared Memory Management: ✓
   SGA size: 3072 MB
   PGA Size: 1024 MB
   ```

### Task 4: Oracle Parameter Configuration

#### 4.1 Connect to Database
```bash
# Switch to oracle user
su - oracle

# Connect to SQL*Plus
sqlplus / as sysdba
```

#### 4.2 Set Display Parameters
```sql
-- Set linesize and pagesize for better output formatting
SQL> set linesize 32000;
SQL> set pagesize 32000;
```

#### 4.3 View Current Parameters
```sql
-- Check current database name and instance
SQL> show parameter name;
SQL> show parameter sga;
SQL> show parameter pga;
```

#### 4.4 Configure Memory Parameters
```sql
-- Configure SGA parameters
SQL> ALTER SYSTEM SET sga_max_size = 3G SCOPE=SPFILE;
SQL> ALTER SYSTEM SET sga_target = 3G SCOPE=SPFILE;

-- Configure PGA parameters
SQL> ALTER SYSTEM SET pga_aggregate_limit = 2G SCOPE=SPFILE;
SQL> ALTER SYSTEM SET pga_aggregate_target = 512M SCOPE=SPFILE;

-- Configure process parameters
SQL> ALTER SYSTEM SET processes = 500 SCOPE=SPFILE;
SQL> ALTER SYSTEM SET open_cursors = 500 SCOPE=SPFILE;
```

#### 4.5 Restart Database to Apply Changes
```sql
-- Shutdown database
SQL> SHUTDOWN IMMEDIATE;

-- Startup database
SQL> STARTUP;
```

#### 4.6 Verify Parameter Changes
```sql
-- Reconnect and set formatting
SQL> set linesize 32000;
SQL> set pagesize 32000;

-- Verify SGA parameters
SQL> show parameter sga;

-- Verify PGA parameters  
SQL> show parameter pga;

-- Verify process parameters
SQL> show parameter processes;

-- Verify cursor parameters
SQL> show parameter cursor;
```

### Task 5: Cold Backup and Database Restart

#### 5.1 Perform Cold Backup
1. **Shutdown Database**
   ```bash
   # Connect as oracle user
   su - oracle
   sqlplus / as sysdba
   ```
   ```sql
   SQL> SHUTDOWN IMMEDIATE;
   ```

2. **Create Cold Backup**
   ```bash
   # Copy Oracle data files to backup location
   cp -r /u01/app/oracle/oradata /tmp/oradata_bkp
   ```

3. **Restart Database**
   ```sql
   # Connect to SQL*Plus
   sqlplus / as sysdba
   
   SQL> STARTUP;
   ```

#### 5.2 Verify Backup
```bash
# List backup contents
oracle > ls -lah /tmp/oradata_bkp/

# Expected output should show:
# - Database files (.dbf)
# - Control files (.ctl)
# - Redo log files (.log)
```

### Task 6: Security Configuration

#### 6.1 Disable SELinux
1. **Edit SELinux Configuration**
   ```bash
   sudo nano /etc/selinux/config
   ```

2. **Modify Configuration**
   ```bash
   # Change from:
   SELINUX=enforcing
   
   # To:
   SELINUX=disabled
   ```

3. **Reboot System**
   ```bash
   sudo reboot
   ```

4. **Verify SELinux Status**
   ```bash
   sestatus
   # Should show: SELinux status: disabled
   ```

#### 6.2 Disable Firewalld
1. **Stop and Disable Firewalld**
   ```bash
   sudo systemctl stop firewalld
   sudo systemctl disable firewalld
   ```

2. **Verify Firewall Status**
   ```bash
   sudo systemctl list-unit-files --type=service | grep firewalld
   # Should show: firewalld.service disabled
   ```

## ⚙️ Configuration

### Database Connection Details
```
Database Name: TESTCDB
PDB Name: TESTPDB
Port: 1521 (default)
Service Name: TESTCDB
SYS/SYSTEM Password: [Set during installation]
```

### Connection Examples
```bash
# Connect to CDB as SYSDBA
sqlplus / as sysdba

# Connect to CDB as SYSTEM
sqlplus system/[password]@//localhost:1521/TESTCDB

# Connect to PDB
sqlplus system/[password]@//localhost:1521/TESTPDB
```

### Optimized Parameters
```sql
-- Memory configuration
sga_max_size: 3G
sga_target: 3G
pga_aggregate_limit: 2G
pga_aggregate_target: 512M

-- Process configuration
processes: 500
open_cursors: 500
```

## 🔒 Security Settings

### Important Security Notes
- **SELinux**: Disabled for development environment (consider enabling for production)
- **Firewall**: Disabled for development environment (configure properly for production)
- **Oracle User**: Use strong passwords and change them regularly
- **Database Passwords**: Use complex passwords and implement password policies

### Production Security Recommendations
```sql
-- Enable password verification
ALTER PROFILE DEFAULT LIMIT PASSWORD_VERIFY_FUNCTION ORA12C_STRONG_VERIFY_FUNCTION;

-- Set password expiration
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME 90;

-- Limit failed login attempts
ALTER PROFILE DEFAULT LIMIT FAILED_LOGIN_ATTEMPTS 5;
```

## 💾 Backup Procedures

### Cold Backup Strategy
```bash
#!/bin/bash
# cold_backup.sh - Automated cold backup script

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/cold_backup_$DATE"
ORACLE_DATA="/u01/app/oracle/oradata"

echo "Starting cold backup at $(date)"

# Shutdown database
sqlplus / as sysdba <<EOF
SHUTDOWN IMMEDIATE;
EXIT;
EOF

# Create backup directory
mkdir -p $BACKUP_DIR

# Copy Oracle data files
cp -r $ORACLE_DATA/* $BACKUP_DIR/

# Restart database
sqlplus / as sysdba <<EOF
STARTUP;
EXIT;
EOF

echo "Cold backup completed at $(date)"
echo "Backup location: $BACKUP_DIR"
```

### Hot Backup with RMAN (Recommended for Production)
```sql
-- Connect to RMAN
rman target /

-- Configure backup settings
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
CONFIGURE DEFAULT DEVICE TYPE TO DISK;
CONFIGURE BACKUP OPTIMIZATION ON;

-- Perform full backup
BACKUP DATABASE PLUS ARCHIVELOG;
```

## 🐛 Troubleshooting

### Common Issues and Solutions

#### 1. Insufficient Memory
**Symptom**: Database startup fails with memory errors
```bash
# Solution: Add swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### 2. Oracle Listener Issues
**Symptom**: Cannot connect to database remotely
```bash
# Check listener status
lsnrctl status

# Restart listener
lsnrctl stop
lsnrctl start

# Check listener configuration
cat $ORACLE_HOME/network/admin/listener.ora
```

#### 3. Database Startup Problems
**Symptom**: Database fails to start
```sql
-- Check alert log
SELECT VALUE FROM V$DIAG_INFO WHERE NAME = 'Diag Trace';

-- Common startup commands
STARTUP NOMOUNT;  -- Start instance only
STARTUP MOUNT;    -- Start and mount
STARTUP;          -- Full startup
```

#### 4. Space Issues
**Symptom**: Tablespace full errors
```sql
-- Check tablespace usage
SELECT 
    tablespace_name,
    ROUND(SUM(bytes)/1024/1024/1024,2) AS size_gb,
    ROUND(SUM(maxbytes)/1024/1024/1024,2) AS max_size_gb
FROM dba_data_files 
GROUP BY tablespace_name;

-- Add datafile to tablespace
ALTER TABLESPACE USERS ADD DATAFILE 
'/u02/oradata/TESTCDB/users02.dbf' SIZE 1G AUTOEXTEND ON;
```

### Log File Locations
```bash
# Oracle Alert Log
tail -f $ORACLE_BASE/diag/rdbms/testcdb/testcdb/trace/alert_testcdb.log

# Listener Log
tail -f $ORACLE_BASE/diag/tnslsnr/$(hostname)/listener/trace/listener.log

# Installation Log
ls -la $ORACLE_BASE/oraInventory/logs/
```

## 📚 Useful Commands

### Database Status Commands
```sql
-- Database information
SELECT NAME, OPEN_MODE, DATABASE_ROLE FROM V$DATABASE;

-- PDB information
SELECT NAME, OPEN_MODE FROM V$PDBS;

-- Memory usage
SELECT * FROM V$MEMORY_TARGET_ADVICE ORDER BY MEMORY_SIZE;

-- Active sessions
SELECT 
    COUNT(*) as session_count,
    STATUS 
FROM V$SESSION 
GROUP BY STATUS;

-- Tablespace usage
SELECT 
    df.tablespace_name,
    ROUND(df.total_space/1024/1024,2) as total_mb,
    ROUND(fs.free_space/1024/1024,2) as free_mb,
    ROUND((df.total_space - fs.free_space)/1024/1024,2) as used_mb,
    ROUND(((df.total_space - fs.free_space)/df.total_space)*100,2) as pct_used
FROM 
    (SELECT tablespace_name, SUM(bytes) as total_space FROM dba_data_files GROUP BY tablespace_name) df,
    (SELECT tablespace_name, SUM(bytes) as free_space FROM dba_free_space GROUP BY tablespace_name) fs
WHERE df.tablespace_name = fs.tablespace_name;
```

### System Performance Monitoring
```bash
# CPU usage by Oracle processes
top -u oracle

# Memory usage
free -h
cat /proc/meminfo | grep -i oracle

# Disk usage
df -h
du -sh /u01/app/oracle/*
du -sh /u02/oradata/*

# Network connections
netstat -an | grep 1521

# Oracle processes
ps -ef | grep oracle
```

### Database Maintenance
```sql
-- Gather statistics
EXEC DBMS_STATS.GATHER_DATABASE_STATS;

-- Check database health
SELECT * FROM V$DATABASE_BLOCK_CORRUPTION;

-- Archive log information
SELECT * FROM V$ARCHIVE_DEST_STATUS;

-- Backup information
SELECT * FROM V$BACKUP_SET;
```

## 📖 References

| Title | Link |
|-------|------|
| Oracle Database 19c Documentation | [View](https://docs.oracle.com/en/database/oracle/oracle-database/19/) |
| Oracle Linux 8 Documentation | [View](https://docs.oracle.com/en/operating-systems/oracle-linux/8/) |
| VirtualBox User Manual | [View](https://www.virtualbox.org/manual/) |
| Oracle Database Installation Guide | [View](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/) |

## 🤝 Contributing

We welcome contributions to improve this installation guide!

### How to Contribute
1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Contribution Guidelines
- Follow the existing documentation format
- Test all commands and procedures
- Include screenshots for complex procedures
- Update the Table of Contents if adding new sections

## 📄 License

This project is for educational purposes. Oracle Database software is a commercial product of Oracle Corporation and is subject to Oracle's licensing terms.

## 👨‍💻 Author

**Toprak Kamburoğlu**
- Department: Computer Engineering
- University: Kadir Has University
- Project: Oracle 19c Installation and Configuration

## 🏢 Organization

<div align="center">
<img src="images/Figes.png" alt="Figes Logo" width="500"/>

This project was developed under the guidance of **Figes** organization.
</div>

## ⚠️ Important Notes

### Development vs Production
- This installation guide is designed for **development and testing environments**
- For **production deployments**, follow Oracle's official documentation and best practices
- Always consult with Oracle DBAs for production configurations

### Security Considerations
- Change all default passwords immediately after installation
- Implement proper backup strategies before going to production
- Configure network security according to your organization's policies
- Enable auditing for production environments

### Version Compatibility
- This guide is specifically for Oracle Database 19c (19.3) Enterprise Edition
- Commands and procedures may vary for different Oracle versions
- Always check Oracle's compatibility matrix for your specific environment

---

<div align="center">

**📝 Note**: This guide provides a comprehensive installation procedure for Oracle Database 19c Enterprise Edition. For different Oracle versions or specific requirements, please refer to Oracle's official documentation.

**⭐ If this guide helped you, please give it a star!**

</div>
