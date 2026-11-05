# Shared Drive Disaster

<escape>**Status:** Completed</escape>

## Project Overview

A simulated Windows company environment designed to investigate, configure, and verify department-based file access using NTFS permissions and PowerShell.

The objective was to ensure that employees could access only the directories authorized for their department, while maintaining controlled cross-department access to a shared directory.

## Environment

- Windows 11 virtual machine
- Local Windows user accounts
- PowerShell
- NTFS access control lists (ACLs)
- Three department groups: HR, IT, and Employees

## Access Control Policy

| Directory | HR | IT | Employees |
|---|---|---|---|
| HR General | Modify | No access | No access |
| HR Confidential | Modify | No access | No access |
| IT General | No access | Modify | No access |
| IT Restricted | No access | Modify | No access |
| Employees Shared | Read | Read | Modify |

## Key Tasks

- Created department-specific local user accounts and security groups.
- Inspected existing NTFS permissions and inheritance.
- Configured explicit group permissions using PowerShell.
- Tested authorized and unauthorized access using separate user accounts.
- Introduced a deliberate permission misconfiguration to simulate unauthorized access.
- Investigated and remediated excessive permissions.
- Verified the final access policy.

## Skills Demonstrated

- Windows user and group administration
- NTFS permissions and ACL management
- PowerShell scripting and administration
- Least privilege
- Access control troubleshooting
- Security verification and documentation

## Repository Structure

- `README.md` — Project overview and summary.
- `investigation-notes.md` — Detailed technical steps, findings, remediation, and verification.
- `screenshots/` — Supporting evidence from the project.

## Outcome

Successfully configured and verified the intended department-level access policy. Confirmed that authorized users could perform their assigned actions while unauthorized users were denied access.
