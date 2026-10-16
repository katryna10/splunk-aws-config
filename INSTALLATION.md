# Splunk Enterprise & Universal Forwarder Installation and Configuration Guide

This document outlines the step-by-step procedures for installing Splunk Enterprise, setting up user configurations, hardening settings, installing the Splunk Universal Forwarder, and linking license slaves to a license master across servers.

---

## Part 1: Installing Splunk Enterprise on Linux Servers

### Step 1: Initial Server Preparation (Root User)
Log in to each server as the root user and prepare the environment:
1. Install `wget` to allow file downloads from the internet[span_0](start_span)[span_0](end_span):
   ```bash
   yum install wget
Create two groups, logreader and admin:
groupadd logreader
groupadd admin
Create a user of your choice:
useradd <your_id>
Add your newly created user to the admin group:
usermod -aG logreader <your_id>
Add the admin group to the sudoers file so members can execute commands with elevated privileges.
Step 2: Register and Download Splunk Enterprise
Go to the Splunk Website and click Free Splunk in the upper right-hand corner.
Fill out personal details using "Linux Admin" for Job Title and "Clearwater" for Company Name.
To download a specific previous release (e.g., version 8.1.0.1) to avoid bugs and ensure version uniformity across servers, visit the Splunk Previous Releases Page.
Select the Linux tab, locate version 8.1.0.1, and click Download Now for the .rpm package (64-bit, kernel Linux distributions).
Click Command Line (wget), then click here to copy the complete wget command.
Step 3: Installation and Initial Setup
Sudo to the root user and navigate to the /tmp directory:
cd /tmp
Paste and execute the copied wget command to download the package.
Install the package using rpm (use [tab] for auto-completion on the long filename):
rpm -ivh splunk-8...
Enable boot start as the root user from /opt/splunk/bin:
./splunk enable boot-start -systemd-managed -user splunk --accept-license --auto-ports --no-prompt
Step 4: Configuring Splunk Enterprise Settings
Switch to the splunk user (sudo su - splunk) to configure local configuration files:

Create Admin Login (user-seed.conf)
File path: /opt/splunk/etc/system/local/user-seed.conf
Content:
[user_info]
USERNAME = admin
PASSWORD = password
Configure Host Name (server.conf)
File path: /opt/splunk/etc/system/local/server.conf
Content:
[general]
serverName = IDX
(Note: Replace student-IDX with the appropriate identifier such as student-UF, student-SH, or student-DS. The pass4SymmKey will auto-generate upon restart).
Configure Disk Usage (server.conf)
File path: /opt/splunk/etc/system/local/server.conf
Content:
[diskUsage]
minFreeSpace = 500
Configure Host Reflection in Logs (inputs.conf)
File path: /opt/splunk/etc/system/local/inputs.conf
Content:
[default]
host = IDX
Enable HTTPS on Web Interface (web.conf)
File path: /opt/splunk/etc/system/local/web.conf
Content:
[settings]
enableSplunkWebSSL = 1
Step 5: Starting Splunk and Finalizing Permissions
Start the Splunk process from /opt/splunk/bin:
./splunk start
Verify access via web browser at https://<yourPUBLICip>:8000 using your user-seed.conf credentials.
Ensure the automatic splunk user is added to the logreader group and that splunk user and group have full ownership over /opt/splunk and /var/log/*.
Part 2: Installing the Splunk Universal Forwarder
Repeat similar steps on two separate servers, noting that the Universal Forwarder is a lightweight agent that does not include a graphical user interface (GUI).

Go to the Splunk Universal Forwarder Previous Releases Page.
Select the Linux tab, choose version 8.1.0.1 (64-bit), and get the wget command for the .rpm package.
Download the package into the /tmp directory using wget and install it using rpm -ivh.
Part 3: Configuring License Master and License Slaves
To control the indexing quota across your deployment, designate one server as the License Master and link all other Splunk Enterprise servers as License Slaves.

Log into the GUI of the server acting as the License Master: https://<PUBLIC_ip_of_server>:8000.
Navigate to Settings > Licensing.
Click Add License and upload your license file, then save.
On all slave servers, run the following command in $SPLUNK_HOME/bin to link them to the master:
./splunk edit licenser-localslave -master_uri 'https://<PRIVATE_ip_of_licensemaster>:8089'
Restart Splunk on the slave servers to apply changes:
./splunk restart
