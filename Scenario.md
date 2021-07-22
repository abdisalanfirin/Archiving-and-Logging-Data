## Archiving and Logging Data
This unit's homework is designed to solidify the following concepts and tools:

Create a tar archive that excludes a directory using the --exclude= command option.
Manage backups using cron jobs.
Write bash scripts to create system resource usage reports.
Perform log filtering using journalctl.
Manage log file sizes using logrotate.
Create an auditing system to check for policy and file violations using auditd.

### Step 1: Create, Extract, Compress, and Manage tar Backup Archives

Command to extract the TarDocs.tar archive to the current directory:

**sudo tar -xvvf TarDocs.tar** 

![Comand to extract](./Images/snap_1.PNG)

Command to create the Javaless_Doc.tar archive from the TarDocs/ directory, while excluding the TarDocs/Documents/Java directory:

**sudo tar cvf Javaless_Docs.tar --exclude=”TarDocs/Documents/Java” TarDocs/**

![Comand to create javeless and exluding java](./Images/snap_2.png)

Command to ensure Java/ is not in the new Javaless_Docs.tar archive:

 **tar tvf MyFinancials.tar | grep -i Documents/Java**

 ![Comand to ensure java is not there](./Images/snap_3.png)
 
 ### Bonus

Command to create an incremental archive called logs_backup_tar.gz with only changed files to snapshot.file for the /var/log directory:

**sudo tar --listed-incremental=snapshot.file -cvzf logs_backup_tar.gz /var/log**

![Comand to create incremental archive](./Images/snap_4.png)

### Critical Analysis Question

Why wouldn't you use the options -x and -c at the same time with tar?

**you can not create and extract a file at the same time.  you need -c create a back file first and then use -x to extract the files.**  



### Step 2: Create, Manage, and Automate Cron Jobs

Cron job for backing up the /var/log/auth.log file:

**0 6 * * 3 tar -zvf /auth_backup.tar.gz /var/log/auth.log**

![Comand to creat cron job](./Images/snap_5.png)

### Step 3: Write Basic Bash Scripts

Brace expansion command to create the four subdirectories:

**sudo mkdir {freemem,diskuse,openlist,freedisk}**

![Comand to creat subdirectories](./Images/snap_6.png)

Paste your system.sh script edits below:

**#!/bin/bash**
**free -h > ~/backups/freemem/free_mem.txt**
**du -h > ~/backups/diskuse/disk_use.txt**
**lsof > ~/backups/openlist/open_list.txt**
**df -h > ~/backups/freedisk/free_disk.txt**

![Comand to set up cript system.sh](./Images/snap_7.png)

Command to make the system.sh script executable:

**chmod +x systtem.sh**

![Comand to exeutable system.sh script](./Images/snap_8.png)

### Optional

Commands to test the script and confirm its execution:

 **sudo ./system.sh**

![Comand to test system.sh script](./Images/snap_9.png)

### Bonus

Command to copy system to system-wide cron directory:

**sudo cp system.sh /etc/cron.d**

![Comand to copy system.sh script](./Images/snap_10.png)

![Comand to copy system.sh script](./Images/snap_x.png)

### Step 4. Manage Log File Sizes

Run sudo nano /etc/logrotate.conf to edit the logrotate configuration file.

Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log.

Add your config file edits below:

**/var/log/auth.log** {
    **weekly**
    **rotate 7**
    **notifempty**
    **delaycompress**
    **missingok**
    **endscript**
**}**

![Comand to add config file](./Images/snap_11.png)

### Bonus: Check for Policy and File Violations

Command to verify auditd is active:

**sudo systemctl status auditd**

![Comand to cgeck the audit is active](./Images/snap_12.png)

Command to set number of retained logs and maximum log file size:

**sudo nano /etc/audit/auditd.conf**

![Comand to cset numbers of retained log](./Images/snap_13.png)

Command using auditd to set rules for /etc/shadow, /etc/passwd and /var/log/auth.log:

**sudo nano /etc/audit/rules.d/audit.rules**

Add the edits made to the rules file below:

**-w /etc/shadow -p wra -k hashpass_audit**
**-w /etc/passwd -p wra -k userpass_audit**
**-w /var/log/auth.log -p wra -k authlog_audit**

![Comand to set rules for /etcshadow and passwd](./Images/snap_14.png)

![Comand to set rules for var log](./Images/snap_14.png)

Command to restart auditd:

**sudo systemctl restart auditd**

![Comand to restart the system](./Images/snap_15.png)

Command to list all auditd rules:

**sudo auditctl -l**

![Comand to list audit rules](./Images/snap_16.png)

Command to produce an audit report:

**sudo aureport -au**

![Comand to short the audit list](./Images/snap_17.png)

Create a user with sudo useradd attacker and produce an audit report that lists account modifications:

**sudo adduser attacker**

![Comand to add user](./Images/snap_18.png)

Command to use auditd to watch /var/log/cron:

**sudo auditctl -w /var/log/cron**

Command to verify auditd rules:

**sudo auditctl -l**

![Comand to verify audit](./Images/snap_19.png)

### Bonus (Research Activity): Perform Various Log Filtering Techniques

Command to return journalctl messages with priorities from emergency to error:

**sudo journalctl -p alert -b**

![Comand to return journalctl](./Images/snap_20.png)

Command to check the disk usage of the system journal unit since the most recent boot:

**sudo journalctl --disk-usage -b 0**

![Comand to check disk usage](./Images/snap_21.png)

Comand to remove all archived journal files except the most recent two:

**sudo journalctl --vacuum-files=2**

![Comand to remove all archived](./Images/snap_22.png)

Command to filter all log messages with priority levels between zero and two, and save output to /home/sysadmin/Priority_High.txt:

**sudo journalctl -p crit >> ~/student/Priority_High.txt**

Command to automate the last command in a daily cronjob. Add the edits made to the crontab file below:

**0 22 * * 1-7 journalctl -p crit >> ~/student/Priority_High.txt**