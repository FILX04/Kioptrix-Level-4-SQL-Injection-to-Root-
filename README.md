📌 Kioptrix Level 4 – SQL Injection to Root (SSH Access + Restricted Shell Escape + MySQL PrivEsc)
📝 Overview

This write‑up documents the full exploitation chain used to compromise the Kioptrix Level 4 vulnerable machine.
The attack path started with a SQL injection vulnerability in the web application, followed by SSH access, restricted shell escape, and a MySQL‑based privilege escalation leading to full root access.

📚 Table of Contents

1. Enumeration

2. Directory Discovery

3. SQL Injection – Authentication Bypass

4. SSH Access Using Leaked Credentials

5. Escaping the Restricted LigGoat Shell

6. Inspecting Web Source Files

7. Privilege Escalation via MySQL UDF

8. Getting Root

9. Root Cause Analysis

10. Mitigation Recommendations

🔍 1. Enumeration

An initial scan was performed to identify open ports and running services.

Open ports included:

SSH (22)

HTTP (80)

SMB (139/445)

📸 Screenshot: Nmap scan results

📂 2. Directory Discovery

A directory brute‑force scan revealed several accessible paths:

/index.php

/member

/logout

/john ← important

📸 Screenshot: Gobuster output

🧨 3. SQL Injection — Authentication Bypass

The main login page was vulnerable to SQL injection.

Using a simple boolean-based payload in the password field allowed bypassing authentication, logging in as john, and revealing valid credentials inside the member control panel.

📸 Screenshot: SQLi payload working
📸 Screenshot: Logged in as John

🔐 4. SSH Access Using Leaked Credentials

With John’s real password obtained, SSH login was successful.

Because SSH used older algorithms, compatibility flags were required.

📸 Screenshot: Successful SSH login

🐚 5. Escaping the Restricted LigGoat Shell

The SSH session dropped into a restricted shell (“LigGoat Shell”) with very limited commands.

A code execution trick was used to break out of the restricted shell and obtain a normal interactive shell.

📸 Screenshot: Shell escape

🕵️ 6. Inspecting Web Source Files

Navigating to /var/www revealed the application source code, including checklogin.php.

This file exposed:

Hardcoded MySQL root username

Empty MySQL root password

Vulnerable SQL query

Commented‑out input sanitization

This confirmed the reason SQL injection was possible.

📸 Screenshot: checklogin.php

⚙️ 7. Privilege Escalation via MySQL UDF

Since MySQL root access required no password, logging in was trivial.

Using MySQL’s sys_exec() function, a system command was executed to add john to the admin group.

📸 Screenshot: sys_exec() abuse

👑 8. Getting Root

After john was added to the admin group, privilege escalation was straightforward:

Running sudo as john

Gaining full root shell

📸 Screenshot: whoami = root

🧵 9. Root Cause Analysis

Primary vulnerabilities identified:

SQL injection in login page

Hard‑coded DB credentials

MySQL root account with no password

Ability to execute system commands via sys_exec()

Outdated Apache/PHP/MySQL stack

Restricted shell escape possible

No proper privilege separation

Attack chain summary:
SQLi → Credential leak → SSH login → Restricted shell escape → MySQL root → sys_exec → Add user to admin → Root

🔒 10. Mitigation Recommendations

Implement prepared statements to prevent SQL injection

Enforce strong DB passwords

Remove dangerous MySQL UDF functions

Update outdated Apache/PHP/SSH/Samba versions

Disable interactive login for web app service accounts

Enforce proper user privilege separation

Limit SSH access and use MFA where possible

🎉 Conclusion

Kioptrix Level 4 demonstrates how a simple SQL injection can snowball into full system compromise when combined with weak system configuration and outdated services.
This exercise reinforces the importance of secure coding, least privilege, and hardening legacy systems.
