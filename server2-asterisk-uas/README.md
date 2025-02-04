# Asterisk 20.11.1 Installation and Rsyslog Configuration Guide

This guide provides step-by-step instructions for installing Asterisk 20.11.1 and configuring Rsyslog on Linux.  

## Why Rsyslog?

We need Rsyslog to capture and manage Asterisk logs effectively.  
Specifically, we'll use it to send the SIP logs to the remote log storage server for further processing.  
Rsyslog offers robust logging capabilities, allowing us to monitor SIP traffic and troubleshoot any issues that may arise.

## Prerequisites

*   A Linux system.
*   Root or sudo privileges.
*   Internet access.

## Asterisk Installation Steps

1.  **Install `chkconfig` (if not already installed):** This is needed for enabling services at boot.

    ```bash
    sudo dnf install chkconfig -y
    ```

2.  **Create Asterisk group and user:**

    ```bash
    groupadd asterisk
    useradd -r -d /var/lib/asterisk -g asterisk asterisk
    ```

3.  **Set correct ownership and permissions:**

    ```bash
    chown -R asterisk.asterisk /etc/asterisk /var/{lib,log,spool}/asterisk /usr/lib64/asterisk
    restorecon -vr {/etc/asterisk,/var/lib/asterisk,/var/log/asterisk,/var/spool/asterisk}
    ```

4.  **Install Asterisk dependencies:**

    ```bash
    contrib/scripts/install_prereq install
    ```

5.  **Configure the build:**

    ```bash
    ./configure --libdir=/usr/lib64 --with-jansson-bundled=yes
    ```

6.  **Run `make menuselect`:** This allows you to select optional modules. If you're unsure, you can usually accept the defaults.

    ```bash
    make menuselect
    ```

7.  **Compile Asterisk:**

    ```bash
    make
    ```

8.  **Install Asterisk:**

    ```bash
    sudo make install
    ```

9.  **Copy the systemd service file:**

    ```bash
    sudo cp /run/systemd/generator.late/multi-user.target.wants/asterisk.service /etc/systemd/system/
    ```

10. **Reload systemd daemon and enable Asterisk service:**

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable asterisk
    ```

11. **Start Asterisk:**

    ```bash
    sudo systemctl start asterisk
    ```

12. **Verify Asterisk status:**

    ```bash
    sudo systemctl status asterisk
    ```

13. **Configure Firewall (firewalld):**

    ```bash
    sudo firewall-cmd --zone=public --add-service sip --permanent
    sudo firewall-cmd --zone=public --add-port=10000-20000/udp --permanent  # For RTP traffic
    sudo firewall-cmd --add-port=5060/tcp --permanent # For SIP over TCP (if used)
    sudo firewall-cmd --add-port=5060/udp --permanent # For SIP over UDP (if used)
    sudo firewall-cmd --reload
    ```

14. **Access Asterisk CLI:**

    ```bash
    asterisk -r
    ```

## Configuration Files (Attached to the Git Repository)

At this point, before moving on to Rsyslog, I've attached the necessary Asterisk configuration files 
(`pjsip.conf`, `extensions.conf`, `asterisk.conf`, `modules.conf`, `logger.conf`) to the Git repository for your reference. 
These files contain the settings I used for my setup. 
You can use them as a starting point, but please remember to adjust them according to your specific needs.

## Rsyslog Installation and Configuration

1.  **Install Rsyslog (if not already installed):**  Rsyslog is likely already installed on Rocky Linux, but if it's not:

    ```bash
    sudo dnf install rsyslog -y
    ```

2.  **Create Rsyslog configuration file for Asterisk:**

    ```bash
    sudo touch /etc/rsyslog.d/asterisk.conf
    sudo vi /etc/rsyslog.d/asterisk.conf
    ```

3.  **Add the following configuration to `/etc/rsyslog.d/asterisk.conf`:**

    ```
[modules]
noload => chan_sip.so 
load => chan_pjsip.so # As we intended to use PJSIP
    ```

4.  **Restart Rsyslog:**

    ```bash
    sudo systemctl restart rsyslog
    ```

5.  **Verify Rsyslog status:**

    ```bash
    sudo systemctl status rsyslog
    ```
6. **Configure rsyslog to send logs to the remote server (3rd server)**
   
    ```bash
    *.* @@<remoteserverIP>:<port>  # Please refer rsyslog.conf
    ```
   
## Important Notes

*   Refer to the official Asterisk and Rsyslog documentation for detailed information on configuration options.
*   The provided firewall rules are basic. You might need to adjust them based on your specific network setup and security requirements.
