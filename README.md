# Active Directory Home Lab

A 6-day hands-on Active Directory home lab built from scratch in **Oracle VirtualBox** using **Windows Server 2022** and **Windows 11**. This repository documents the full build—from promoting a fresh Domain Controller and configuring an Enterprise Root CA to managing OUs, least-privilege file shares, Group Policy troubleshooting, and PowerShell user-provisioning automation.

---

## Lab Environment

| Component | Configuration | Role in Lab |
| :--- | :--- | :--- |
| **`DC01`** | Windows Server 2022 Standard Evaluation (`10.0.2.3/24`, DNS `127.0.0.1`) | Primary Domain Controller hosting AD DS, DNS, AD CS (`LAB-DC01-CA`), and SMB shares |
| **`WS01`** | Windows 11 Pro client | Domain-joined workstation used to test user logons, file shares, and GPO enforcement |
| **Virtual Network** | VirtualBox NAT Network `ADnetwork` (`10.0.2.0/24`) | Isolated virtual subnet connecting `DC01` and `WS01` |
| **Forest / Domain** | `LAB.local` | Single-forest, single-domain Active Directory environment |

---

## Quick Navigation

| Day | Focus Area | Key Topics | Walkthrough |
| :---: | :--- | :--- | :--- |
| **Day 1** | Domain Controller & AD CS | Server rename, AD DS install, `LAB.local` forest promotion, Enterprise Root CA | [View Day 1 (11 screenshots)](./day-01-domain-controller/README.md) |
| **Day 2** | Users & Built-In Groups | ADUC, `Groups` OU, Domain Admin scope, domain user naming conventions | [View Day 2 (5 screenshots)](./day-02-domain-users/README.md) |
| **Day 3** | Networking & Domain Join | VirtualBox NAT network (`ADnetwork`), static IP & self-DNS, `WS01` join | [View Day 3 (4 screenshots)](./day-03-domain-join/README.md) |
| **Day 4** | OUs & File Share Access | `IT` / `Marketing` / `Administrators` OUs, `IT-Share` group, NTFS/SMB least privilege | [View Day 4 (10 screenshots)](./day-04-ous-and-file-shares/README.md) |
| **Day 5** | GPO Troubleshooting | Account Lockout Policy, GPO link order precedence, `gpresult` / `rsop.msc` | [View Day 5 (3 screenshots)](./day-05-gpo-lockout-policy/README.md) |
| **Day 6** | PowerShell Automation | `Get-ADUser`, `Get-ADOrganizationalUnit`, `New-ADUser`, CSV bulk onboarding in VS Code | [View Day 6 (8 screenshots)](./day-06-powershell-automation/README.md) |

---

## 6-Day Project Log

### Day 1 — Windows Server 2022 Domain Controller & AD CS Setup

Today I set up my first Windows Server 2022 Domain Controller from scratch in VirtualBox:

- Renamed the server
- Installed AD DS
- Promoted it to a DC, and
- Created my first forest/domain

Also configured Active Directory Certificate Services with an Enterprise Root CA.

1. Renamed the server to `DC01`
2. Installed the AD DS role
3. Configured a new forest (`LAB.local`)
4. Prerequisites check — all passed
5. Logged in as domain admin — promotion confirmed
6. Installed AD CS role
7. Configured AD CS successfully

Next up: joining a client machine to the domain.

→ [View Day 1 Walkthrough & Screenshots (11)](./day-01-domain-controller/README.md)

---

### Day 2 — Active Directory Users and Computers

Today I focused on Active Directory Users and Computers:

- Created an OU to organize built-in groups
- Reviewed the built-in `Administrator` account and confirmed it's the Domain Admin (should only be used when domain-level changes are needed — not for everyday tasks)
- Created my first domain user accounts with a consistent naming convention

**Key Lesson:** In a real environment, you'd never set "password never expires" or skip forced password changes the way I did here for lab convenience — good reminder of the gap between lab shortcuts and production security practices.

Next up: attaching a Windows 11 client to the domain and logging in as one of these users.

→ [View Day 2 Walkthrough & Screenshots (5)](./day-02-domain-users/README.md)

---

### Day 3 — Connecting the Domain Controller and Windows 11 Client

Today I connected the two machines together:

- Created a dedicated NAT network so my domain controller and client could communicate
- Set a static IP on the DC with DNS pointing to itself
- Joined my Windows 11 client (`WS01`) to the domain
- Logged in using a domain user account created back on Day 2 — instead of the local admin account

Small but important detail: logging in with the actual domain user (not the built-in `Administrator`) matters here — it's the same principle real IT support follows: use the least-privileged account needed for the task.

Verified the join from the domain controller side too — `WS01` now shows up under Active Directory's `Computers` container.

Next up: Organizational Units, Group Policy, and PowerShell automation.

→ [View Day 3 Walkthrough & Screenshots (4)](./day-03-domain-join/README.md)

---

### Day 4 — Organizational Units, Security Groups & Least-Privilege Access

Today was about structure and access control:

- Created Organizational Units to mirror a real company's departments (`Marketing`, `IT`)
- Nested an `Administrators` OU inside `IT` to separate privileged accounts from regular staff
- Created a security group (`IT-Share`) and added specific users to it
- Set up a shared folder on the server (`\\DC01\IT-Share-Folder`), removed default access, and granted permissions only to that group
- Tested it two ways: logged in as a group member (worked ✅), then as a non-member (correctly denied ❌)

This is a good example of the principle of least privilege in practice — access isn't given by default, it's explicitly granted to only the people who need it.

Next up: Group Policy Objects.

→ [View Day 4 Walkthrough & Screenshots (10)](./day-04-ous-and-file-shares/README.md)

---

### Day 5 — Group Policy Objects (The Troubleshooting Edition)

Set out to configure an Account Lockout Policy via Group Policy: lock an account after 3 failed login attempts, then unlock it from the domain controller.

The policy didn't apply on the first try. Traced it through a few layers:

- Account Lockout Policy only takes effect when linked at the domain root — not on individual OUs
- Even after moving it to the domain root, it still didn't fire, because `Default Domain Policy` had higher link order precedence and was silently overriding it with its default "never lock out" value
- `gpresult` and `rsop.msc` both needed to be run elevated to actually show Computer Configuration data — without that, it looked like nothing was applying at all
- Fixed it by configuring the lockout threshold directly inside `Default Domain Policy` instead of a separate competing GPO — removing the conflict entirely

Tested it end to end: locked an account after 3 wrong attempts, then unlocked it from Active Directory Users and Computers.

This ended up being a much better lesson than the original plan — GPO precedence and inheritance are exactly the kind of thing that trips people up in real environments, and now I've actually debugged it firsthand.

→ [View Day 5 Walkthrough & Screenshots (3)](./day-05-gpo-lockout-policy/README.md)

---

### Day 6 — Automating Active Directory with PowerShell

Moved from clicking through the GUI to managing Active Directory with PowerShell:

- Queried existing users and OUs directly from the command line
- Created a single user account with `New-ADUser`, understanding each parameter (`SamAccountName`, `UserPrincipalName`, `Path`, `SecureString` password)
- Built a CSV of multiple new hires and wrote a script to bulk-create all of them in one run, instead of clicking "New User" repeatedly

This is the real difference between GUI work and scripting — creating 3 users by hand is fine, but creating 50 for a new office opening isn't something anyone does by clicking through a wizard fifty times. This is where IT starts to scale.

Used VS Code with the PowerShell extension to write and reuse the script, rather than typing one-off commands into a terminal.

→ [View Day 6 Walkthrough & Screenshots (8)](./day-06-powershell-automation/README.md)

---

## Key Takeaways & Real-World Lessons

- **Principle of Least Privilege:** Separate standard daily user accounts from Domain Admin privileges, nest privileged admin OUs, and remove default inheritance on sensitive shares before granting explicit group permissions.
- **Lab Shortcuts vs. Production Security:** Setting "Password never expires" or skipping change-at-next-logon is convenient for lab testing, whereas production environments enforce rotation and onboarding password resets (`-ChangePasswordAtLogon $true`).
- **GPO Precedence & Diagnostics:** Password and account lockout policies must be linked at the domain root (and reconciled with `Default Domain Policy`), and diagnostic tools like `gpresult` and `rsop.msc` require elevation to inspect Computer Configuration settings.
- **Scaling with Automation:** PowerShell (`Import-Csv` + `New-ADUser`) turns repetitive GUI onboarding tasks into repeatable, auditable workflows.
