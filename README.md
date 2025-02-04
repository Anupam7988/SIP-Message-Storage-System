# SIP-Message-Storage-System: Real-time SIP Transaction Analysis and Storage

This system provides a comprehensive solution for real-time analysis and storage of SIP (Session Initiation Protocol) transaction data. It captures, processes, and stores SIP messages, enabling efficient monitoring, troubleshooting, and in-depth analysis of VoIP communications.

## System Overview

The system is designed for real-time insights into SIP traffic, leveraging a modular and scalable architecture. It consists of three key components working in concert:

1.  **SIPp (Server 1 - UAC):** SIPp simulates concurrent SIP calls, acting as a User Agent Client (UAC). This controlled traffic generation allows for realistic testing and load analysis of the system.

2.  **Asterisk (Server 2 - UAS):** Asterisk functions as the User Agent Server (UAS), receiving and processing SIP requests. It generates detailed SIP transaction logs, the core data source for the system. PJSIP (`chan_pjsip.so`) is used as the SIP stack in Asterisk due to its modern features, active maintenance, and superior performance compared to the deprecated `chan_sip.so`. Logs are stored in `/var/log/asterisk/pjsip` and rotated using cron jobs for efficient disk management.

3.  **Logstash and MySQL (Server 3 - Data Processing and Storage):** This server handles the processing and persistent storage of SIP log data. Asterisk forwards logs to this server in real-time using Rsyslog, a high-performance log processing tool. Logstash then parses and structures the raw logs using Grok filters, grouping related messages by `Call-ID`. Finally, the processed data is stored in a MySQL database.

## Data Flow and Processing

The flow of data through the system is as follows:

1.  **SIPp generates SIP calls:** Simulating real-time VoIP traffic.
2.  **Asterisk processes calls and generates logs:** Stored in `/var/log/asterisk/pjsip`.
3.  **Rsyslog forwards logs:** To the third server for centralized processing.
4.  **Logstash parses logs:** Using Grok to group transactions by `Call-ID`.
5.  **Data stored in MySQL:** For efficient querying and analysis.

## Log Processing with Logstash and Grok

Logstash uses Grok patterns (defined in the Logstash configuration file) to parse the raw SIP logs. Grok allows us to define patterns that match the structure of SIP messages. By targeting the `Call-ID`, we can group all messages belonging to the same call transaction.

## MySQL Database Schema

The MySQL database stores the processed SIP transaction data. The schema is optimized for querying by `Call-ID` and includes:

*   `call_id`: Unique identifier for each SIP session.
*   `sip_method`: SIP method (e.g., INVITE, ACK, BYE, REGISTER).
*   `from`: SIP address of the caller.
*   `to`: SIP address of the callee.
*   `cseq`: Command Sequence Number.
*   `sdp_content`: Session Description Protocol (SDP) data.

## Ports and Firewall Configuration

*   **SIPp:** SIPp communicates with Asterisk on port 5060 (UDP and TCP).
*   **Asterisk:** Asterisk listens on port 5060 (UDP and TCP) for SIP traffic and uses ports 10000-20000 (UDP) for RTP.
*   **Rsyslog:** Rsyslog uses port 514 (UDP and TCP) for log forwarding.
*   **MySQL:** MySQL listens on port 3306.

Ensure that your firewall allows traffic on these ports.  For example, using `firewalld`:  `sudo firewall-cmd --permanent --add-port={5060/udp,5060/tcp,10000-20000/udp,514/udp,514/tcp,3306/tcp} && sudo firewall-cmd --reload` (Adapt as needed for your firewall solution).

## Database Credentials

Configure the MySQL database username and password in the Logstash configuration file (`logstash_sip_logs.conf`). Use strong and secure credentials.

## Log Rotation

Log rotation is configured using cron jobs to periodically archive older logs and prevent excessive disk space usage. The frequency of log rotation and the number of log files kept can be configured in the cron job settings.

## Advantages of this System

*   **Real-time SIP call tracking:** Enables quick identification of call failures and efficient troubleshooting.
*   **Centralized logging:** Ensures all SIP transactions are captured and stored reliably.
*   **Automated log processing:** Logstash and Grok automate the processing of large log volumes.
*   **Efficient data storage and retrieval:** MySQL provides a structured way to query and analyze SIP data.
*   **Scalability and maintainability:** The modular design allows the system to adapt to changing needs.

## Installation and Configuration

Detailed installation and configuration guides for each component (SIPp, Asterisk, Rsyslog, Logstash, and MySQL) are provided in separate README files within their respective directories. All necessary configuration files are included in this repository. Remember to adjust paths, credentials, and IP addresses to match your environment.

## Usage

1.  Start SIPp.
2.  Start Asterisk.
3.  Start Logstash and MySQL.
4.  Query the MySQL database to analyze SIP transactions.

## TESTING ON A LIVE SYSTEM

The testing has been shown in the `TESTING ON A LIVE SYSTEM.md` file attached to the GIT.
