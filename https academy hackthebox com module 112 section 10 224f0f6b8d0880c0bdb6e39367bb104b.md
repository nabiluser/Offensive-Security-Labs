# https://academy.hackthebox.com/module/112/section/1067

### **Title**

**Sensitive Information Disclosure via Anonymous SMB Access**

### **Summary**

The SMB service on the target server allows anonymous access (Null Session) to a share named `sambashare`. This misconfiguration allows an unauthenticated attacker to list, read, and download sensitive files. During the assessment, a specific flag file (`flag.txt`) was retrieved, and user configuration files (`.bashrc`, `.profile`) were identified, revealing that the share is mapped to a user's home directory.

### **Steps to Reproduce**

**Step 1:**
Scan the target IP to identify open ports using Nmap.
Command: `nmap -sV -sC -Pn -p139,445 10.129.171.81`

**Step 2:**
List the available SMB shares allowing anonymous login.
Command: `smbclient -N -L //10.129.171.81`

**Step 3:**
Connect to the exposed `sambashare` without a password.
Command: `smbclient -N //10.129.171.81/sambashare -L` 

**Step 4:**
Navigate through the directories, list hidden files to identify the system path, and download the `flag.txt` file.

### **4) Proof of Concept (PoC)**

**Screenshots:**

- **Screenshot 1:** Nmap scan results showing ports 445,139 are open.

![image.png](image.png)

- **Screenshot 2:** Listing shares using `smbclient` showing `sambashare` and the comment "InFreight SMB v3.1".
    
    ![image.png](image%201.png)
    
- **Screenshot 3:** Successful login to `sambashare` and listing files including `.profile` and `.bashrc`.

![image.png](image%202.png)

- **c< 4:** Downloading and reading the `flag.txt`

![image.png](image%203.png)

![image.png](image%204.png)

**Technical Analysis (Path Discovery):**
Upon accessing the share, standard Linux user configuration files were found:

Plaintext

  `.profile             H      807  Tue Feb 25 07:03:22 2020
  .bashrc              H     3771  Tue Feb 25 07:03:22 2020`

The presence of these hidden files indicates that the share is directly mapped to a user's home directory. Based on Linux standard conventions and the share name context, the full system path is identified as:
**`/home/sambauser`**

**Retrieved Data:**
File: `flag.txt`
Content: `HTB{o873nz4xdo873n4zo873zn4fksuhldsf}`

### **5) Impact**

An unauthenticated attacker can access and steal sensitive internal data.

- **Information Leakage:** The attacker retrieved the "flag" proof.
- **System Enumeration:** Revealing the full system path (`/home/sambauser`) and configuration files helps the attacker understand the internal file structure and potentially find hardcoded credentials or aliases in `.bashrc`, which facilitates further privilege escalation attacks.

### **Recommendation**

**Short fix:**
Disable anonymous/guest access to the `sambashare` directory and ensure proper permission bits are set.