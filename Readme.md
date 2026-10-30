# Stored Cross-Site Scripting (Stored XSS) in Device/Policy/Script/Client/Site Name Management Leading to Unauthorized Execution in Admin Session and Mass Account Takeover

**Description:** The application fails to properly sanitize user input and encode output for input fields including device name, policy name, script, client, and site. When a user inputs malicious HTML/JavaScript payloads into these fields, the data is stored directly in the database. 
The malicious code automatically executes in the browser of any user (especially Administrators/Technicians) who views the management list or interacts with dropdown menus containing these compromised objects.

**Impact:**
- **Account Takeover (ATO)**
  - **Impersonation:** An attacker can append the stolen `access_token` to API headers to impersonate the victim and perform arbitrary actions, such as updating personal details, changing passwords, or adding/removing 2FA configurations.
  - **Authentication Bypass:** Bypasses perimeter security controls (strong passwords, OTP/2FA) entirely because the exfiltrated token represents a valid, active session.
- **Critical Infrastructure Threat (RMM Impact)**
  - **Mass Device Disruption:** If the compromised account holds administrator privileges over the RMM platform, the attacker can leverage management APIs to send shutdown, reboot, network disconnect, or service termination commands across all endpoints/agents managed by the account.
  - **Mass Remote Code Execution (RCE):** Through the RMM's automated Script/Command execution feature, an attacker can deploy malware, ransomware, or malicious scripts to wipe data across the entire fleet of devices with a single command.
  - **Availability Impact (High):** Causes severe disruption to enterprise business operations and halts core IT infrastructure services.
- **Privilege Escalation**
  - **Impact:** If the victim triggered by the payload is an Admin/Super Admin, the attacker implicitly gains full administrative control over the entire RMM system without needing additional privilege escalation steps.

**Vulnerable Endpoints**

| Endpoint | Parameter |
| :--- | :--- |
| POST /clients/ | `name` |
| PUT /clients/ | `name` |
| PUT /clients/sites/ | `name` |
| POST /automation/policies/ | `name` |
| PUT /automation/policies/ | `name` |
| POST /scripts/ | `name` |
| PUT /scripts/ | `name` |

**Proof of Concept (PoC):**

In this attack scenario, an attacker uses the account `longnbh_2` (a low-privileged user) to inject a Stored XSS payload into a client name. When the victim (`longnbh` - an administrator) opens the client selection dropdown in the Audit Log feature, the payload triggers and exfiltrates the victim's Authorization token to an attacker-controlled webhook, allowing the attacker to completely hijack the administrator account.

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103703.png)
![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103743.png)

**Step 1: Injecting the Stored XSS Payload**

To set a client name containing the Stored XSS payload, we can either create a new client or edit an existing one. In this PoC, we update an existing client name.

**Edit Client Request (`PUT /clients/`)**

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103801.png)
![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103809.png)

When the victim accesses the Audit Log and opens the client dropdown, a popup appears confirming that the XSS script executed successfully.

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103818.png)

Using the same vector, instead of a simple `alert()` script, an attacker can replace it with a malicious payload designed to exfiltrate the user's Authorization token upon execution.

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103825.png)

**Step 2: Victim Interacts with the Client Dropdown**

The admin user (`longnbh`) accesses the Audit Log and opens the client selection dropdown menu.

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103835.png)
![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103856.png)

**Step 3: Modifying Password & 2FA to Complete Account Takeover**

Using the intercepted token, the attacker submits requests to change the victim's account password and reset 2FA, completing the account takeover.

![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103919.png)
![image](https://github.com/Kuri119/Stored-XSS-In-Tactical-RMM-1.5.2/blob/main/POC/Screenshot%202026-10-30%20103930.png)

With both the password and 2FA updated, the attacker gains full control of the `longnbh` admin account.

**Note:** Similar payload injection vectors exist across the creation and modification endpoints for Sites, Scripts, and Policies. The payload executes whenever a user interacts with a dropdown menu containing these infected items.
