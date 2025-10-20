# Windows_server_2022
Isolated VMware lab exercise: Kali → Windows Server 2022. Documented safe reconnaissance and WinRM fingerprinting (port 5985), credentialed enumeration in lab-only environment, remediation recommendations, and lessons learned. For educational use only — no exploit code or real credentials included.

# Example curl (sanitized)
curl -v --negotiate -u : \
  -H "Content-Type: application/soap+xml;charset=UTF-8" \
  --data-binary @identify.xml \
  "http://LAB_HOST:5985/wsman"

# Evil-WinRM (sanitized)
evil-winrm -i LAB_IP -u LAB_USER -p 'REDACTED_PASSWORD'

# Inside Evil-WinRM / remote PowerShell
ls
cd LAB_FOLDER
mkdir LAB_FILE.txt

-------------------------------------------------------------------------------------------------------------------------
Here is the professional, organized text based on your table format, suitable for a "readme" file or detailed documentation:

***

## Windows Server 2022 Exploitation via WinRM

This document details the step-by-step process of enumerating and gaining remote access to a target Windows Server 2022 machine using the **Windows Remote Management (WinRM)** service in a controlled lab environment.

### Phase 1: Service Identification and Fingerprinting

The initial steps were focused on confirming the service running on the target and gathering its authentication requirements.

1.  **Initial Host Discovery:**
    * **Command:** `nmap -R Victime Ip`
    * **Purpose:** Confirmed the host was reachable and active on the network.
2.  **Service Version Enumeration:**
    * **Command:** `nmap -sV -p 5985 --script=banner,http-title victime_ip`
    * **Purpose:** Verified that port **5985** was open and running the **WinRM (HTTP)** service.
3.  **Protocol Check (GET Request):**
    * **Command:** `curl -v http://victimeIP:5985/wsman`
    * **Purpose:** Confirmed the WinRM endpoint protocol. The service typically responds by prompting for a POST request.
4.  **Target Configuration Analysis (Internal Check):**
    * **Commands:** `winrm get winrm/config` or `Get-ChildItem -Path WSMan:\localhost\Service \| Format-List *`
    * **Purpose:** These commands (executed on the target server itself) were used to check the WinRM listener configuration, including enabled authentication methods (e.g., NTLM, Kerberos).

### Phase 2: WinRM Identify Requests

The SOAP **Identify** request was used to gather server details and determine authentication requirements.

1.  **Unauthenticated POST (Identify):** A `curl` command was executed with a SOAP `Identify` payload to the `/wsman` endpoint.
    * **Purpose:** To test for anonymous identification and reveal server information.
    * **Outcome Analysis:** A **HTTP/1.1 401 Unauthorized** response was expected, and the presence of `WWW-Authenticate: Negotiate, NTLM` headers confirmed that authentication was mandatory, and the server accepted NTLM and Kerberos.
2.  **NTLM Authenticated POST:**
    * **Command:** A `curl` command using the `--ntlm -u 'user:pass'` flags was used.
    * **Purpose:** To verify valid domain credentials against the WinRM service. A successful attempt yields a **200 OK** response with a detailed IdentifyResponse XML.

### Phase 3: Successful Remote Access and Exploitation

With validated credentials, a dedicated remote access tool was utilized to establish an interactive shell.

1.  **Tool Selection (evil-winrm):** As NTLM/Kerberos authentication was required and available, the **evil-winrm** Ruby-based tool was used for a more stable connection than basic `curl` commands.
2.  **Exploitation Command:**
    * `evil-winrm -i victimeIP -u Administrator -p 'YourPassword'`
    * **Purpose:** This command established an interactive **remote PowerShell session** on the Windows Server 2022 as the Administrator user, successfully completing the exploitation phase.

***

### Hardening and Defensive Recommendations

These steps are crucial for securing the server tested in the lab:

* **Restrict Access:** Implement firewall rules to limit access to WinRM ports **(5985/5986)** only from trusted administrative IP addresses.
* **Use HTTPS Only:** **Disable the insecure HTTP listener (Port 5985)** and exclusively enable **WinRM over HTTPS (Port 5986)** to ensure all remote management traffic is encrypted.
* **Strong Authentication:** Ensure administrative accounts utilize strong passwords and, ideally, multi-factor authentication (MFA).



