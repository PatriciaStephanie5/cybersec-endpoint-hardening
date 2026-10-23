# Systems Hardening & Security Baseline Lab Report

---

## 1. Executive Summary & Lab Objectives

The primary objective of this lab is to establish, verify, and document a robust security baseline across endpoint operating systems, network services, access control frameworks, and wireless infrastructure. 

The lab follows a structured, continuous security lifecycle:
1. **Identify Weaknesses:** Scan for unpatched packages, unencrypted administrative protocols (e.g., Telnet), weak authentication settings, and exposed network attack surfaces.
2. **Apply Security Controls:** Implement OS patch management, enforce Group Policies, deploy SSH encryption, mandate file encryption (EFS), enforce Role-Based Access Control (RBAC), and isolate wireless guest clients.
3. **Verify Configurations:** Validate active SSH connectivity, test local group permissions, verify encrypted files, and confirm Wi-Fi segmentation via virtual management interfaces.
4. **Document Improved Posture:** Maintain detailed documentation and proof-of-concept evidence for compliance tracking and audit readiness.

---

## 2. Secure Remote Management: SSH Implementation on Kali Linux

### Baseline Rationale
Non-Windows assets such as Linux servers, firewalls, and Cisco routers are primarily administered via Command Line Interfaces (CLI). Legacy management protocols such as Telnet or unencrypted HTTP transmit credentials and session data across the network in cleartext, exposing the system to credential sniffing, man-in-the-middle (MitM) attacks, and packet injection. 

To eliminate cleartext exposure, **Secure Shell (SSH)** is deployed to provide end-to-end asymmetric encryption, strong session integrity, and secure command execution over untrusted networks.

---

### Step-by-Step Installation & Service Configuration

1. **Resolve Package Dependencies & Update System:**
   ```bash
   sudo apt update
   sudo apt --fix-broken install
   sudo apt full-upgrade -y
   ```

2. **Install OpenSSH Server:**
   ```bash
   sudo apt install openssh-server
   ```

3. **Start & Enable the SSH Service:**
   ```bash
   sudo service ssh start
   ```

---

### Verification & Proof of Concept

#### Evidence 1: Terminal SSH Installation & Service Activation
As captured in the installation output, the `openssh-server` package was unpacked and configured successfully. The SSH daemon service was subsequently started on the Kali host via `sudo service ssh start`:

check files uploaded in this repository

#### Evidence 2: Remote Connection via PuTTY Client
A remote session was initiated from a Windows host (`192.168.1.7`) using the **PuTTY SSH client**. The session successfully authenticated as user `kali`, traversed into the user's Desktop directory (`/Desktop`), and opened the target test directory (`sshtest`):

check files uploaded in this repository

```bash
kali@kali:~$ cd Desktop
kali@kali:~/Desktop$ ls
sshtest
kali@kali:~/Desktop$ cd sshtest
kali@kali:~/Desktop/sshtest$
```

---

## 3. Windows Endpoint Hardening & Group Policy Enforcement

To harden local Windows machines against unauthorized access and privilege escalation, Local Group Policy (`gpedit.msc`) is leveraged to enforce administrative security policies across three key pillars:

### A. Password & Account Policies
* **Password Complexity Requirements:** Enforces a minimum password length (12+ characters) along with a combination of uppercase, lowercase, numbers, and special characters to mitigate brute-force and dictionary attacks.
* **Account Lockout Threshold:** Automatically locks user accounts after a set number of failed authentication attempts (e.g., 5 invalid attempts), stopping automated password spraying.
* **Password Age & History:** Restricts password reuse across multiple rotation cycles.

### B. Role-Based Access Control (RBAC) & Least Privilege
* **Principle of Least Privilege (PoLP):** Configures security groups and explicit Discretionary Access Control Lists (DACLs) so users operate strictly with the minimum necessary access rights required for their roles.
* **Group Management:** Assigns users to functional groups (e.g., *Auditors*, *Administrators*, *Standard Users*) rather than assigning direct per-user permissions, eliminating permission sprawl.

### C. Data Protection via Encrypting File System (EFS)
* **File-Level Encryption:** Applies EFS (`cipher.exe` or local folder properties) to sensitive target directories and files.
* **At-Rest Protection:** Protects confidential files against offline access or physical disk theft by tying decryption keys directly to the authenticated Windows user profile.

---

## 4. Wireless Network Security & Architecture

Using the TP-Link AX3200 Wi-Fi 6 Router emulator interface (`https://emulator.tp-link.com`), baseline wireless hardening was configured to minimize attack surface exposure.

### Key Concepts & Defensive Controls

1. **Attack Surface:** The sum of all accessible endpoints, open protocols, broadcast SSIDs, and unencrypted channels where an unauthorized user can attempt to extract data or gain unauthorized access.
2. **Wireless Encryption Standards (WPA2 / WPA3):**
   * **WPA2 (AES-CCMP):** Replaces legacy, vulnerable encryption methods (WEP/WPA).
   * **WPA3-Personal (SAE):** Implements Simultaneous Authentication of Equals (SAE) to prevent offline dictionary attacks and enforce Forward Secrecy.
3. **Wireless Authentication:** Distinguishes between Pre-Shared Key (WPA2/WPA3-Personal) for smaller segments and WPA-Enterprise (802.1X/RADIUS) for centralized user identity management.

---

### Configuration & Proof of Concept

#### Evidence 1: Corporate Wireless Baseline Configuration
The primary corporate network (`2.4GHz` / `5GHz`) is configured with modern **WPA2/WPA3-Personal** hybrid security, Wi-Fi performance settings (OFDMA, TWT), and strong passphrase protection:

check files uploaded in this repository

#### Evidence 2: Isolated Guest Network Configuration
A dedicated **Guest Network** (`TP-Link_Guest_22DD`) is enabled on the `2.4GHz` band to segment external devices:

check files uploaded in this repository

* **Network Isolation:** Guest clients are strictly segregated from the internal corporate network and local broadcast domains, preventing rogue guest devices from probing internal subnets or executing lateral movement attacks.
* **Guest Authentication:** Uses WPA2/WPA3-Personal encryption with a dedicated guest password (`tplinkpassword`).

---

## 5. Security Posture Summary Matrix

| Security Domain | Identified Weakness / Vulnerability | Applied Security Control | Verification Method |
| :--- | :--- | :--- | :--- |
| **Linux Remote Admin** | Cleartext management risk (Telnet/HTTP) | Package updates & OpenSSH Server deployment (`openssh-server`) | Successful PuTTY SSH connection to Kali Linux desktop |
| **Account Security** | Weak passwords & brute-force vulnerability | Enforced Password Complexity & Account Lockout Threshold via Local Group Policy | Policy audit via `gpedit.msc` and failed login test |
| **Access Control** | Excessive user privileges & permission sprawl | Role-Based Access Control (RBAC) & Least Privilege group mapping | DACL verification on sensitive system folders |
| **Data Protection** | Plaintext files vulnerable to offline extraction | Encrypting File System (EFS) folder encryption | EFS attribute verification on local target directories |
| **Wireless Infrastructure** | Rogue client interception & lateral network intrusion | WPA2/WPA3-Personal encryption & Isolated Guest SSID | TP-Link AX3200 virtual emulator configuration |
