# SIPp v3.3 Installation Guide

This guide provides step-by-step instructions for installing SIPp version 3.3 from scratch on a Linux system.

## Prerequisites

Before you begin, ensure you have the following prerequisites:

- A Linux system (this guide assumes a Debian/Ubuntu-based distribution, but the steps can be adapted for other distributions).
- Basic command-line knowledge.
- Internet access to download the SIPp source code and dependencies.

## Installation

SIPp can be easily installed using the `apt` package manager.

### 1. Update the package list:
```bash
sudo apt update
```

### 2. Install SIPp:
```bash
sudo apt install sipp -y
```

### 3. Install Dependencies:
```bash
sudo apt install build-essential g++ libpcap-dev libssl-dev -y
```

### 4. Check the SIPp version:
```bash
sipp -v
```

### 5. Navigate to the directory containing your `uac.xml` script:  
   (Or create a `.xml` file if it doesn't exist)

### 6. Alter the script as per the requirement to perform concurrent calls:
```bash
sudo vi /path/uac.xml
```

### 7. Run the script:
```bash
sipp -sn uac -r 1 -l 1 -d 5000 -s <user_extension> <asteriskIP>:<Asterisk_Server_Port>
```

> **Note:**  
> `<Asterisk_Server_Port>`: Replace with the SIP port of your Asterisk server (usually `5060`).

