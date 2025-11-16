# Network Security Assessment and Reconnaissance Report

## Project Summary

This project documents the process of conducting a basic security assessment, focusing on network reconnaissance, port scanning, and initial vulnerability research. The workflow followed a structured 8-step process using industry-standard tools: Nmap for discovery and Wireshark for verification.

**Status:** Completed
**Target Environment:** Local Network (192.168.1.0/24)

---

## 🎯 Task Workflow & Execution

The following steps were executed to perform the assessment:

1.  **Install Nmap** from the official website.
2.  **Find local IP range** (e.g., 192.168.1.0/24).
3.  **Run Nmap:** `sudo nmap -sS -sV -O -T4 192.168.1.0/x` (TCP SYN, Service/Version detection, OS detection, aggressive timing).
4.  **Note down IP addresses and open ports found.**
5.  **Analyze packet capture with Wireshark.**
6.  **Research common services running on those ports.**
7.  **Identify potential security risks from open ports.**
8.  **Save scan results as a text or HTML file.**

---

## 🔎 Key Findings & Analysis

The scan identified multiple hosts. The most critical findings were related to the main network gateway and a target host running a non-standard web service.

### Host: 192.168.x.x (RTKGW.bbrouter) - Primary Gateway

| Port | State | Service | Version | Assessment |
| :--- | :--- | :--- | :--- | :--- |
| **21/tcp** | **open** | **ftp** | GNU Inetutils FTPd 1.4.x | FTP traffic is unencrypted (plaintext). |
| **80/tcp** | **open** | **http** | Boa HTTPd 0.93.xx | Unencrypted service. **Critical:** Running an extremely outdated version. |
| **443/tcp** | **open** | **ssl/http** | Boa HTTPd 0.93.xx | Encrypted service, but the underlying server (Boa HTTPd 0.93.xx) is vulnerable. |

### Host: 192.168.1.xx (kali.bbrouter) - Specific Target

To verify the handshake status on this host, a Wireshark capture was performed, filtered on `tcp.port == 80 || tcp.port == 443`.

#### Wireshark Verification (Task 5)

The packet analysis confirms the state of the ports through the TCP handshake process:

| Port | Status | Evidence (TCP Flag Analysis) |
| :--- | :--- | :--- |
| **TCP Port 80** | **OPEN** | A SYN packet was answered with a **SYN, ACK** from the target (192.168.1.x), completing the two-way handshake. |
| **TCP Port 443** | **CLOSED** | A SYN packet was answered with a **RST, ACK** from the target (192.168.1.x), indicating an explicit rejection. |
---

## 🚨 Security Risks & Recommendations (Tasks 6–7)

### 1. **CRITICAL RISK:** Outdated Web Server

* **Vulnerability:** The Boa HTTPd 0.93.xx web server software is decades old and contains publicly known vulnerabilities (e.g., buffer overflows, remote code execution flaws).
* **Recommendation:** **IMMEDIATELY** patch or replace the web server software with a modern, maintained alternative (e.g., a current version of Apache or Nginx).

### 2. **HIGH RISK:** Unencrypted Services

* **Vulnerability:** Services running on Port 80 (HTTP) and Port 21 (FTP) transmit sensitive data and credentials in plaintext.
* **Recommendation:**
    * **Port 80:** Enforce a **HTTPS redirect** (Port 443) to encrypt all web traffic.
    * **Port 21:** Disable FTP and migrate to **SFTP** (Secure File Transfer Protocol) over SSH.
