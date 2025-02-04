# Logstash and MySQL Installation Guide for SIP Message Storage

This guide details the installation and configuration of Logstash and MySQL for storing and processing SIP messages.  


## Prerequisites

*   A Linux System.
*   Root or sudo privileges.
*   Internet access.


## Rsyslog Configuration for Remote Logging (Optional but Recommended)

We need to receive logs from the remote Asterisk server which is why need to configure Rsyslog to receive remote logs:

1.  **Install Rsyslog (if not already installed):**

    ```bash
    sudo dnf install rsyslog -y
    ```

2.  **Create directory for remote logs:**

    ```bash
    sudo mkdir -p /var/log/asterisk_remote
    sudo chown root:root /var/log/asterisk_remote  # Set appropriate ownership
    sudo chmod 755 /var/log/asterisk_remote
    ```

3.  **Edit Rsyslog configuration (`/etc/rsyslog.conf` or `/etc/rsyslog.d/remote_log_receiver.conf`):** Add the following lines to enable UDP reception:

    ```
    module(load="imudp") # Load the UDP input module
    input(type="imudp" port="514") # Listen on UDP port 514 (default syslog port)
    ```

4.  **Create Rsyslog rule to store Asterisk logs from the remote host:**

    ```bash
    sudo vi /etc/rsyslog.d/asterisk_remote.conf
    ```

5.  **Add the following configuration to `/etc/rsyslog.d/asterisk_remote.conf`:**

    ```
    if (fromhost-ip == '192.168.29.146') then { # Replace with the IP of the remote Asterisk server
        action(type="omfile" file="/var/log/asterisk_remote/pjsip.log")
        stop # Stop processing further rules for these messages
    }
    ```

6.  **Restart Rsyslog:**

    ```bash
    sudo systemctl restart rsyslog
    ```

7.  **Enable Rsyslog on boot:**

    ```bash
    sudo systemctl enable rsyslog
    ```

8.  **Configure Firewall (firewalld) to allow remote syslog (port 514):**

    ```bash
    sudo firewall-cmd --permanent --add-port=514/udp
    sudo firewall-cmd --add-port=514/tcp  # If using TCP
    sudo firewall-cmd --reload
    ```

## MySQL Installation and Setup

1.  **Install MySQL server:**

    ```bash
    sudo dnf install mysql-server -y
    ```

2.  **Start and enable MySQL service:**

    ```bash
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    ```

3.  **Secure MySQL installation:** Run the `mysql_secure_installation` script to set a root password, remove anonymous users, disallow remote root login, and remove the test database.

    ```bash
    sudo mysql_secure_installation
    ```

4.  **Install Python MySQL connector:**

    ```bash
    sudo dnf install python3-pip -y  # If pip isn't already installed
    pip3 install mysql-connector-python
    ```

## Logstash Installation and Configuration

1.  **Install Java (required by Logstash):**

    ```bash
    sudo dnf install java-11-openjdk -y
    ```

2.  **Add the Elastic repository:**

    ```bash
    sudo rpm --import [https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch)
    sudo nano /etc/yum.repos.d/elastic.repo
    ```

    Add the following content to the `elastic.repo` file:

    ```
    [logstash-7.x]
    name=Elastic Logstash 7.x
    baseurl=[https://artifacts.elastic.co/packages/7.x/yum](https://www.google.com/search?q=https://artifacts.elastic.co/packages/7.x/yum)
    gpgcheck=1
    gpgkey=[https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch)
    enabled=1
    ```

3.  **Install Logstash:**

    ```bash
    sudo dnf install logstash -y
    ```

4.  **Install the Logstash JDBC plugin:**

    ```bash
    sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-jdbc
    ```

5.  **Create Logstash configuration file (`/etc/logstash/conf.d/sip_logs.conf`):**  The configuration file I used is attached to the Git repository (`logstash_sip_logs.conf`). You'll need to adjust the JDBC connection string, username, password, and database name to match your MySQL setup.

6.  **Start and enable Logstash service:**

    ```bash
    sudo systemctl start logstash
    sudo systemctl enable logstash
    ```

7.  **Test Logstash configuration (optional but recommended):**

    ```bash
    sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/sip_logs.conf
    ```

8.  **Restart Logstash:**

    ```bash
    sudo systemctl restart logstash
    ```

## Configuration Files (Attached to the Git Repository)

I've attached the following configuration files to the Git repository for your convenience:

*   `asterisk.conf`: Main Asterisk configuration.
*   `logstash_sip_logs.conf`: Logstash configuration for processing SIP logs and sending them to MySQL.
*   `mysql-server.cnf`: MySQL server configuration.
*   `mysql_schema.sql`: SQL schema for the MySQL database to store SIP data.
*   `rsyslog.conf`: Rsyslog configuration (if you are not using remote logging, you may not need this file).

Remember to modify these files according to your specific setup, especially the database credentials and paths.

