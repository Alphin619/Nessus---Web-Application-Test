# Nessus - Metasploitable 2 Web Application Tests

## 📌 Overview
Metasploitable 2 is an intentionally vulnerable virtual machine designed for security testing. Nessus is a vulnerability scanner used to identify weaknesses across systems, services, and web applications.

This project focuses on performing **web application vulnerability testing** against the DVWA application hosted on Metasploitable 2 using Nessus.

---

## 🧪 Lab Environment
- **Scanner:** Kali Linux (Nessus Essentials)
- **Target:** Metasploitable 2
- **Target IP:** `192.168.1.4`
- **Web Application:** DVWA (`/dvwa/login.php`)

---

## 🔍 <ins>Target Discovery</ins>

#### **1️⃣ Identify Target IP**
On the Metasploitable VM: `ifconfig`  
<img width="329" height="82" alt="Screenshot 2025-12-21 152620" src="https://github.com/user-attachments/assets/924dc743-b8d4-4518-93a0-bdaa44777845" />

#### 2️⃣ Verify Connectivity
From Kali Linux: `ping 192.168.1.4`  
<img width="261" height="100" alt="Screenshot 2025-12-21 153115" src="https://github.com/user-attachments/assets/a2617c8c-75b1-41ef-b500-55ec022f0d75" />

Successful responses confirmed the host was reachable.

### 🚀<ins> Nessus Setup</ins>
`sudo systemctl start nessusd`  
`sudo systemctl status nessusd`  
Service status: Active  
<img width="368" height="117" alt="Screenshot 2025-12-22 150532" src="https://github.com/user-attachments/assets/a28eade1-3865-45af-bdd0-fd8c258f8c62" />

Access Nessus via browser: `https://127.0.0.1:8834`


### ⚙️<ins> Web Application Scan Configuration</ins>
Scan Type
Template: Web Application Tests  
Scan Name: Meta Application Scan  
Target: 192.168.1.4  
<img width="359" height="195" alt="Screenshot 2025-12-28 145702" src="https://github.com/user-attachments/assets/b8395d56-6620-4f54-b903-13632d31e057" />

Advanced Settings
Scan Type: Scan for all web vulnerabilities (complex)  
<img width="407" height="204" alt="Screenshot 2025-12-28 152014" src="https://github.com/user-attachments/assets/d455bde9-fdbb-4851-8b41-b05079028e55" />

Credentialed Web Authentication

Credentials → HTTP  
Username: `admin`  
Password: `password`  
Login URL: `/dvwa/login.php`  
<img width="462" height="269" alt="Screenshot 2025-12-28 152929" src="https://github.com/user-attachments/assets/775e6bc4-72cd-44ba-b3bb-1046bcfa0b24" />
<img width="829" height="126" alt="Screenshot 2025-12-28 153221" src="https://github.com/user-attachments/assets/299e2e00-1662-471d-b5b8-0ea8950dbe20" />

This allows Nessus to authenticate into the web application and perform authenticated web vulnerability testing.

Save the configuration and launch the scan.

### 📊 <ins>Scan Results</ins>
Total vulnerabilities detected: 43

<img width="812" height="319" alt="Screenshot 2025-12-28 170118" src="https://github.com/user-attachments/assets/b2dafe0c-43e5-4d8e-9b16-883d173bc653" />

These findings include web misconfigurations, insecure PHP settings, and critical remote code execution risks.
<img width="513" height="116" alt="Screenshot 2025-12-28 170223" src="https://github.com/user-attachments/assets/fbeb257e-c225-47ca-be75-b11207ebfc1a" />
<img width="510" height="111" alt="Screenshot 2025-12-28 170239" src="https://github.com/user-attachments/assets/604cf37d-cdec-4d79-91a9-3b58dbbb655b" />


### 🚨 <ins>Example of one Critical Finding: Apache PHP-CGI Remote Code Execution</ins>

One of the most severe findings was:  
Apache PHP-CGI Remote Code Execution  
CVE: <ins>CVE-2012-1823</ins>  
<img width="639" height="266" alt="Screenshot 2025-12-28 170648" src="https://github.com/user-attachments/assets/05352c06-c8df-4c82-8c3c-17df6c727d60" />

The exploit payload extracted from Nessus output and decoded revealed the following parameters:

<img width="517" height="320" alt="Screenshot 2025-12-28 170726" src="https://github.com/user-attachments/assets/becd1868-7329-4a96-bf28-a1f40ff64db8" />

`-d allow_url_include=on`  
`-d safe_mode=off`  
`-d suhosin.simulation=on`  
`-d disable_functions=""`  
`-d open_basedir=none`  
`-d auto_prepend_file=php://input`  
`-d cgi.force_redirect=0`  
`-d cgi.redirect_status_env=0`  
`-n`

### 🔬 <ins>Breakdown of Exploit Parameters</ins>
allow_url_include=on
→ Enables remote file inclusion

safe_mode=off
→ Disables PHP security restrictions

suhosin.simulation=on
→ Puts Suhosin in log-only mode (no blocking)

disable_functions=""
→ Re-enables dangerous PHP functions (exec, system, etc.)

open_basedir=none
→ Removes filesystem access restrictions

auto_prepend_file=php://input
→ Executes PHP code from HTTP request body

cgi.force_redirect=0
→ Disables CGI redirect protection

cgi.redirect_status_env=0
→ Disables CGI environment validation

-n
→ Ignores all php.ini configuration files

<ins>This configuration completely disables PHP security mechanisms.</ins>

### ⚠️ <ins>Impact Assessment</ins>
This vulnerability enables:

<ins>Remote file inclusion</ins>  
<ins>Arbitrary PHP code execution</ins>  
<ins>Execution of OS-level commands</ins>  
<ins>Full filesystem access</ins>  
<ins>Payload injection via HTTP request body</ins>

#### Reference:

https://nvd.nist.gov/vuln/detail/CVE-2012-1823
### 🧠 <ins>SOC Detection Context</ins>

If this payload is observed in:

Logs → <ins>Server is likely under active exploitation</ins>

Scripts → <ins>Indicates a planted backdoor</ins>

URLs → <ins>Confirmed exploit attempt</ins>

### 📝 <ins>Summary</ins>
This web application scan highlights how misconfigured PHP environments can lead to full system compromise. Authenticated web testing provided deeper visibility into application-level risks that would not be visible through network scans alone.

<ins>This lab reinforces the importance of secure web configuration, credentialed scanning, and prioritizing critical RCE vulnerabilities in SOC and vulnerability management workflows.</ins>

--- 
