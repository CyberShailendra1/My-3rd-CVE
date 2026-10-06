# CVE-2026-105703: Improper Authorization in Password Change (Admin Panel)

- **Researcher:** Shailendra Mourya ([CyberShailendra](https://github.com/CyberShailendra1))
- **Contact:** cybershailendra1@gmail.com
- **CVE ID:** CVE-2026-105703
- **Entry:** VDB-413696
- **Product:** User Registration & Login and User Management System With admin panel
- **Vendor:** [PHPGurukul](https://phpgurukul.com/user-registration-login-and-user-management-system-with-admin-panel/)
- **Version:** V3.3
- **Vulnerability Type:** CWE-863 / CWE-287 / CWE-306 (Incorrect Authorization / Broken Access Control)
- **CVSS Score:** 4.7 (Medium Severity)

---

## 📌 Description

A vulnerability was determined in **PHPGurukul User Registration & Login and User Management System 3.3**. The impacted element is an unknown function of the file `loginsystem/admin/change-password.php` of the component **Change Password Handler**. This manipulation of the argument `currentpassword` causes incorrect authorization. Remote exploitation of the attack is possible. The exploit has been publicly disclosed and may be utilized.

---

## 🔬 Analysis
* 10/06/2026*

The PHPGurukul User Registration, Login, and User Management System version 3.3 contains a critical authentication bypass vulnerability within its administrative change password functionality. This flaw resides in the `loginsystem/admin/change-password.php` file, specifically affecting the function responsible for validating user credentials during password modification operations. 

The core technical issue stems from improper validation of the `currentpassword` argument provided by the client. Instead of rigorously verifying that the supplied current password matches the stored hash or credential data associated with the authenticated session, the system fails to enforce this check correctly. This logical error allows an attacker to manipulate the input parameter in a way that bypasses the authentication gate. Consequently, the application proceeds with the password change operation without confirming the identity of the requester, leading to incorrect authorization decisions where unauthorized entities can assume control over administrative accounts by simply providing arbitrary valid credentials.

From an operational perspective, this vulnerability poses a severe risk to system integrity and confidentiality because it enables remote exploitation. By sending specially crafted HTTP requests to the change-password endpoint, malicious actors can take over administrative accounts and establish persistence. This undermines the fundamental trust model of the application, allowing attackers to escalate privileges, exfiltrate sensitive data stored within the user management database, and potentially pivot to further attacks against the backend infrastructure.

The vulnerability aligns with:
- **CWE-863:** Incorrect Authorization
- **CWE-287:** Improper Authentication
- **CWE-306:** Missing Authentication for Critical Function
- **MITRE ATT&CK:** Initial Access & Privilege Escalation (Account Takeover / Credential Manipulation)

---

## 💥 Impact

1. **Re-Authentication & Verification Bypass:**
   - The password change functionality relies on verifying the user's old password as a security barrier. This flaw completely bypasses that authentication requirement.

2. **Session Hijacking & Permanent Account Takeover:**
   - If an attacker gains temporary access to an admin session, they can change the password without knowing the actual current password of that account, permanently locking out the legitimate administrator.

3. **Multi-Admin Escalation:**
   - In environments with multiple administrators, knowing or supplying credentials from any one admin account allows modifying passwords across other active administrator sessions.

---

## 🚀 Proof of Concept (PoC)

1. Authenticate to the Admin Dashboard (`/loginsystem/admin/`).
2. Navigate to the **Change Password** module (`/loginsystem/admin/change-password.php`).
3. Send a POST request where `currentpassword` is set to any valid password found in the database `admin` table:

```http
POST /loginsystem/admin/change-password.php HTTP/1.1
Host: target-host
Cookie: PHPSESSID=<valid_admin_session>
Content-Type: application/x-www-form-urlencoded

currentpassword=TargetAdminPassword&newpassword=NewPassword123!&update=Update
```

4. The backend validates the password against the database without verifying ownership against `$_SESSION['adminid']`, and successfully updates the active administrator's password.

---

## 🛡️ Remediation

To mitigate this vulnerability, immediate remediation is required:
1. **Enforce Ownership Validation:** Implement strict server-side validation ensuring `currentpassword` is verified specifically against the authenticated session user's ID (`$_SESSION['adminid']`) using prepared statements.
2. **Modern Security Standards:** Adhere to OWASP Authentication Cheat Sheet guidelines.
3. **Defense in Depth:** Enforce multi-factor authentication (MFA) for administrative functions and implement rate-limiting on sensitive endpoints to prevent automated abuse.

---

## 📜 References

- **CVE ID:** CVE-2026-105703
- **VulDB Entry:** VDB-413696
- **Vendor:** [PHPGurukul](https://phpgurukul.com/)
- **CWE Classifications:** [CWE-863](https://cwe.mitre.org/data/definitions/863.html), [CWE-287](https://cwe.mitre.org/data/definitions/287.html), [CWE-306](https://cwe.mitre.org/data/definitions/306.html)
