# SIP-Message-Storage-System: Real-time SIP Transaction Analysis and Storage

This system provides a comprehensive solution for real-time analysis and storage of SIP (Session Initiation Protocol) transaction data. It captures, processes, and stores SIP messages, and builds a database system to retrieve the sip call. 

## System Overview

The system is designed for real-time insights into SIP traffic, leveraging a modular and scalable architecture. It consists of three key components working in concert:

1. **SIPp (Server 1 - UAC):** SIPp simulates concurrent SIP calls, acting as a User Agent Client (UAC). This controlled traffic generation allows for realistic testing and load analysis of the system.

2. **Asterisk (Server 2 - UAS):** Asterisk functions as the User Agent Server (UAS), receiving and processing SIP requests. It generates detailed SIP transaction logs, the core data source for the system. PJSIP (`chan_pjsip.so`) is used as the SIP stack in Asterisk due to its modern features, active maintenance, and superior performance compared to the deprecated `chan_sip.so`. Logs are stored in `/var/log/asterisk/pjsip` and rotated using cron jobs for efficient disk management.

3. **Logstash and MySQL (Server 3 - Data Processing and Storage):** This server handles the processing and persistent storage of SIP log data. Asterisk forwards logs to this server in real-time using Rsyslog, a high-performance log processing tool. Logstash then parses and structures the raw logs using Grok filters, grouping related messages by `Call-ID`. Finally, the processed data is stored in a MySQL database.

![Architecture Diagram](https://raw.githubusercontent.com/Anupam7988/SIP-Message-Storage-System/main/architecture.png)

## Data Flow and Processing

The flow of data through the system is as follows:

1. **SIPp generates SIP calls:** Simulating real-time VoIP traffic.
2. **Asterisk processes calls and generates logs:** Stored in `/var/log/asterisk/pjsip`.
3. **Rsyslog forwards logs:** To the third server for centralized processing.
4. **Logstash parses logs:** Using Grok to group transactions by `Call-ID`.
5. **Data stored in MySQL:** For efficient querying and analysis.

## Log Processing with Logstash and Grok

Logstash uses Grok patterns (defined in the Logstash configuration file) to parse the raw SIP logs. Grok allows us to define patterns that match the structure of SIP messages. By targeting the `Call-ID`, we can group all messages belonging to the same call transaction.

## MySQL Database Schema

The MySQL database stores the processed SIP transaction data. The schema is optimized for querying by `Call-ID` and includes:

* `call_id`: Unique identifier for each SIP session.
* `sip_method`: SIP method (e.g., INVITE, ACK, BYE, REGISTER).
* `from`: SIP address of the caller.
* `to`: SIP address of the callee.
* `cseq`: Command Sequence Number.
* `sdp_content`: Session Description Protocol (SDP) data.

## Ports and Firewall Configuration

* **SIPp:** SIPp communicates with Asterisk on port 5060 (UDP and TCP).
* **Asterisk:** Asterisk listens on port 5060 (UDP and TCP) for SIP traffic and uses ports 10000-20000 (UDP) for RTP.
* **Rsyslog:** Rsyslog uses port 514 (UDP and TCP) for log forwarding.
* **MySQL:** MySQL listens on port 3306.

Ensure that your firewall allows traffic on these ports. For example, using `firewalld`:

```bash
sudo firewall-cmd --permanent --add-port={5060/udp,5060/tcp,10000-20000/udp,514/udp,514/tcp,3306/tcp} && sudo firewall-cmd --reload
```
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

## Testing on a Live System

**Process:**

1.  Start SIPp and perform concurrent calls.
2.  Start Asterisk.
3.  Start Logstash and MySQL.
4.  Query the MySQL database to analyze SIP transactions.

**Test Setup:**

1.  A test user has been added in Asterisk and SIPp. (Extension configuration, authentication, and dial patterns are present in the `.conf` file attached to the repository).

    *   From: SIPp > exten = `sipp`
    *   To: Asterisk > exten = `1000`

2.  Log in to SIPp and execute the following command (this command sets the call rate to 10 calls per second. SIPp will attempt to initiate 10 new calls every second):

    ```bash
    sipp -sn uac -r 10 -l 10 -d 5000 -s 1000 192.168.29.146:5060
    ```

    Example SIPp output:

    ```
    ------------------------------ Scenario Screen -------- [1-9]: Change Screen --
    Call-rate(length)    Port    Total-time      Total-calls     Remote-host
    10.0(5000 ms)/1.000s    5060            114.22 s        230     192.168.29.146:5060(UDP)

    0 new calls during 0.013 s period          0 ms scheduler resolution
    10 calls (limit 10)                        Peak was 10 calls, after 1 s
    1 Running, 72 Paused, 0 Woken up
    0 dead call msg (discarded)                0 out-of-call msg (discarded)
    3 open sockets

                                    Messages        Retrans         Timeout         Unexpected-Msg
        INVITE ---------->          230             0               0               
        100 <----------          230             0               0               0
        180 <----------          0               0               0               0
        183 <----------          0               0               0               0
        200 <----------  E-RTD1 230             0               0               0
        ACK ---------->          230             0               
        Pause [   5000ms]          230             
        BYE ---------->          220             0               0               
        200 <----------          220             0               0               0

    ------------------------------ Test Terminated --------------------------------

    ----------------------------- Statistics Screen ------- [1-9]: Change Screen --
    Start Time                     | 2025-02-04      10:32:46:000    1738665166.000360
    Last Reset Time                | 2025-02-04      10:34:40:212    1738665280.212504
    Current Time                   | 2025-02-04      10:34:40:227    1738665280.227550
    -------------------------+---------------------------+--------------------------
    Counter Name                   | Periodic value            | Cumulative value
    -------------------------+---------------------------+--------------------------
    Elapsed Time                   | 00:00:00:015            | 00:01:54:227
    Call Rate                      |   0.000 cps               |   2.014 cps
    -------------------------+---------------------------+--------------------------
    Incoming call created          |           0               |           0
    OutGoing call created          |           0               |         230
    Total Call created             |                           |         230
    Current Call                   |          10               |               
    -------------------------+---------------------------+--------------------------
    Successful call                |           0               |         220
    Failed call                    |           0               |           0
    -------------------------+---------------------------+--------------------------
    Response Time 1                | 00:00:00:000            | 00:00:00:005
    Call Length                    | 00:00:00:000            | 00:00:05:009
    ------------------------------ Test Terminated --------------------------------
    ```

3.  Go to Asterisk CLI (`asterisk -rvv`) and enable PJSIP logging (`CLI> pjsip set logger on`).  Live calls can be seen in the CLI.

4.  Once SIPp call emulation is ended, the logs will be stored in `/var/log/asterisk/pjsip`.  For example, a sample call with `Call-ID: 9-3973@127.0.1.1` will generate logs similar to the following:

```sql  
Feb  1 18:32:31 localhost asterisk[6541]: INVITE sip:1000@192.168.29.146:5060 SIP/2.0
Feb  1 18:32:31 localhost asterisk[6541]: Via: SIP/2.0/UDP 127.0.1.1:5060;branch=z9hG4bK-3973-9-0
Feb  1 18:32:31 localhost asterisk[6541]: From: sipp <sip:sipp@127.0.1.1:5060>;tag=3973SIPpTag009
Feb  1 18:32:31 localhost asterisk[6541]: To: 1000 <sip:1000@192.168.29.146:5060>
Feb  1 18:32:31 localhost asterisk[6541]: Call-ID: 9-3973@127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: CSeq: 1 INVITE
Feb  1 18:32:31 localhost asterisk[6541]: Contact: sip:sipp@127.0.1.1:5060
Feb  1 18:32:31 localhost asterisk[6541]: Max-Forwards: 70
Feb  1 18:32:31 localhost asterisk[6541]: Subject: Performance Test
Feb  1 18:32:31 localhost asterisk[6541]: Content-Type: application/sdp
Feb  1 18:32:31 localhost asterisk[6541]: Content-Length:   129
Feb  1 18:32:31 localhost asterisk[6541]: #015
Feb  1 18:32:31 localhost asterisk[6541]: v=0
Feb  1 18:32:31 localhost asterisk[6541]: o=user1 53655765 2353687637 IN IP4 127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: s=-
Feb  1 18:32:31 localhost asterisk[6541]: c=IN IP4 127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: t=0 0
Feb  1 18:32:31 localhost asterisk[6541]: m=audio 6000 RTP/AVP 0
Feb  1 18:32:31 localhost asterisk[6541]: a=rtpmap:0 PCMU/8000


Feb  1 18:32:31 localhost asterisk[6541]: SIP/2.0 100 Trying
Feb  1 18:32:31 localhost asterisk[6541]: Via: SIP/2.0/UDP 127.0.1.1:5060;rport=5060;received=192.168.29.154;branch=z9hG4bK-397
3-9-0
Feb  1 18:32:31 localhost asterisk[6541]: Call-ID: 9-3973@127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: From: "sipp" <sip:sipp@127.0.1.1>;tag=3973SIPpTag009
Feb  1 18:32:31 localhost asterisk[6541]: To: "1000" <sip:1000@192.168.29.146>
Feb  1 18:32:31 localhost asterisk[6541]: CSeq: 1 INVITE
Feb  1 18:32:31 localhost asterisk[6541]: Server: Asterisk PBX 20.11.1
Feb  1 18:32:31 localhost asterisk[6541]: Content-Length:  0
Feb  1 18:32:31 localhost asterisk[6541]: #015


Feb  1 18:32:31 localhost asterisk[6541]: SIP/2.0 200 OK
Feb  1 18:32:31 localhost asterisk[6541]: Via: SIP/2.0/UDP 127.0.1.1:5060;rport=5060;received=192.168.29.154;branch=z9hG4bK-397
3-9-0
Feb  1 18:32:31 localhost asterisk[6541]: Call-ID: 9-3973@127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: From: "sipp" <sip:sipp@127.0.1.1>;tag=3973SIPpTag009
Feb  1 18:32:31 localhost asterisk[6541]: To: "1000" <sip:1000@192.168.29.146>;tag=c4a3d618-664b-4984-8789-a82b0285c12e
Feb  1 18:32:31 localhost asterisk[6541]: CSeq: 1 INVITE
Feb  1 18:32:31 localhost asterisk[6541]: Server: Asterisk PBX 20.11.1
Feb  1 18:32:31 localhost asterisk[6541]: Contact: <sip:192.168.29.146:5060>
Feb  1 18:32:31 localhost asterisk[6541]: Allow: OPTIONS, REGISTER, SUBSCRIBE, NOTIFY, PUBLISH, INVITE, ACK, BYE, CANCEL, UPDATE, PRACK, INFO, MESSAGE, REFER
Feb  1 18:32:31 localhost asterisk[6541]: Supported: 100rel, timer, replaces, norefersub
Feb  1 18:32:31 localhost asterisk[6541]: Content-Type: application/sdp
Feb  1 18:32:31 localhost asterisk[6541]: Content-Length:   183
Feb  1 18:32:31 localhost asterisk[6541]: #015
Feb  1 18:32:31 localhost asterisk[6541]: v=0
Feb  1 18:32:31 localhost asterisk[6541]: o=- 53655765 2353687639 IN IP4 192.168.29.146
Feb  1 18:32:31 localhost asterisk[6541]: s=Asterisk
Feb  1 18:32:31 localhost asterisk[6541]: c=IN IP4 192.168.29.146
Feb  1 18:32:31 localhost asterisk[6541]: t=0 0
Feb  1 18:32:31 localhost asterisk[6541]: m=audio 17492 RTP/AVP 0
Feb  1 18:32:31 localhost asterisk[6541]: a=rtpmap:0 PCMU/8000
Feb  1 18:32:31 localhost asterisk[6541]: a=ptime:20
Feb  1 18:32:31 localhost asterisk[6541]: a=maxptime:140
Feb  1 18:32:31 localhost asterisk[6541]: a=sendrecv


Feb  1 18:32:31 localhost asterisk[6541]: ACK sip:1000@192.168.29.146:5060 SIP/2.0
Feb  1 18:32:31 localhost asterisk[6541]: Via: SIP/2.0/UDP 127.0.1.1:5060;branch=z9hG4bK-3973-9-5
Feb  1 18:32:31 localhost asterisk[6541]: From: sipp <sip:sipp@127.0.1.1:5060>;tag=3973SIPpTag009
Feb  1 18:32:31 localhost asterisk[6541]: To: 1000 <sip:1000@192.168.29.146:5060>;tag=c4a3d618-664b-4984-8789-a82b0285c12e
Feb  1 18:32:31 localhost asterisk[6541]: Call-ID: 9-3973@127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: CSeq: 1 ACK
Feb  1 18:32:31 localhost asterisk[6541]: Contact: sip:sipp@127.0.1.1:5060
Feb  1 18:32:31 localhost asterisk[6541]: Max-Forwards: 70
Feb  1 18:32:31 localhost asterisk[6541]: Subject: Performance Test
Feb  1 18:32:31 localhost asterisk[6541]: Content-Length: 0
Feb  1 18:32:31 localhost asterisk[6541]: #015


Feb  1 18:32:31 localhost asterisk[6541]: BYE sip:sipp@127.0.1.1:5060 SIP/2.0
Feb  1 18:32:31 localhost asterisk[6541]: Via: SIP/2.0/UDP 127.0.0.1:5060;rport;branch=z9hG4bKPj68d17990-e3c0-4cc4-aeb1-8f5c680d7de6
Feb  1 18:32:31 localhost asterisk[6541]: From: "1000" <sip:1000@192.168.29.146>;tag=c4a3d618-664b-4984-8789-a82b0285c12e
Feb  1 18:32:31 localhost asterisk[6541]: To: "sipp" <sip:sipp@127.0.1.1>;tag=3973SIPpTag009
Feb  1 18:32:31 localhost asterisk[6541]: Call-ID: 9-3973@127.0.1.1
Feb  1 18:32:31 localhost asterisk[6541]: CSeq: 4113 BYE
Feb  1 18:32:31 localhost asterisk[6541]: Reason: Q.850;cause=16
Feb  1 18:32:31 localhost asterisk[6541]: Max-Forwards: 70
Feb  1 18:32:31 localhost asterisk[6541]: User-Agent: Asterisk PBX 20.11.1
Feb  1 18:32:31 localhost asterisk[6541]: Content-Length:  0
Feb  1 18:32:31 localhost asterisk[6541]: #015
```

5.  Verify log transfer to the log storage server by running `tail -f /var/log/pjsip` on both the Asterisk and log storage servers.

6.  Run the Logstash script to parse and push the data into MySQL (the script is currently set to run at 5-minute intervals):

    ```bash
    sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/sip_logs.conf
    ```
Markdown

7. **Querying the MySQL Database**

Log in to MySQL and navigate to the database.  To retrieve information for a specific call, use the following SQL query, replacing `'9-3973@127.0.1.1'` with the desired `Call_ID`:

```sql
SELECT * FROM sip_transactions WHERE Call_ID = '9-3973@127.0.1.1';
Example output (for Call_ID = '9-3973@127.0.1.1'):

+----+---------------------+------------+------------------------------------------+------------------------------------------+-------+----------------------------------------------------------------------------------------------------------------------+------------------------+
| id | call_id             | sip_method | from_sip                                  | to_sip                                    | cseq  | sdp_content                                                                                                          | timestamp              |
+----+---------------------+------------+------------------------------------------+------------------------------------------+-------+----------------------------------------------------------------------------------------------------------------------+------------------------+
|  1 | 9-3973@127.0.1.1   | INVITE      | sipp <sip:sipp@127.0.1.1:5060>;tag=3973... | 1000 <sip:1000@192.168.29.146:5060>        | 1     | v=0<br>o=user1 53655765 2353687637 IN IP4 127.0.1.1<br>s=-<br>c=IN IP4 127.0.1.1<br>t=0 0<br>m=audio 6000 RTP/AVP 0<br>a=rtpmap:0 PCMU/8000 | 2025-02-01 18:32:31   |
|  2 | 9-3973@127.0.1.1   | 100 Trying | From: "sipp" <sip:sipp@127.0.1.1>;tag=3973SIPpTag009 | To: "1000" <sip:1000@192.168.29.146> | 1     | NULL                                                                                                                   | 2025-02-01 18:32:31   |
|  3 | 9-3973@127.0.1.1   | 200 OK      | "sipp" <sip:sipp@127.0.1.1>;tag=3973SIPpTag009 | "1000" <sip:1000@192.168.29.146>;tag=c4a3d618... | 1     | v=0<br>o=- 53655765 2353687639 IN IP4 192.168.29.146<br>s=Asterisk<br>c=IN IP4 192.168.29.146<br>t=0 0<br>m=audio 17492 RTP/AVP 0<br>a=rtpmap:0 PCMU/8000<br>a=ptime:20<br>a=maxptime:140<br>a=sendrecv | 2025-02-01 18:32:31   |
|  4 | 9-3973@127.0.1.1   | ACK         | sipp <sip:sipp@127.0.1.1>;tag=3973SIPpTag009 | 1000 <sip:1000@192.168.29.146>;tag=c4a3d618... | 1     | NULL                                                                                                                   | 2025-02-01 18:32:31   |
|  5 | 9-3973@127.0.1.1   | BYE         | 1000 <sip:1000@192.168.29.146>;tag=c4a3d618... | sipp <sip:sipp@127.0.1.1>;tag=3973SIPpTag009   | 4113  | NULL                                                                                                                   | 2025-02-01 18:32:31   |
+----+---------------------+------------+------------------------------------------+------------------------------------------+-------+----------------------------------------------------------------------------------------------------------------------+------------------------+
