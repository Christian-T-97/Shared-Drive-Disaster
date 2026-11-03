
# Shared Drive Disaster — Windows Access Control & Troubleshooting

## Project Overview

This project simulates a company environment where employees need access to shared files and department-specific documents based on their job responsibilities.

The objective is to configure Windows user accounts, security groups, and NTFS permissions to enforce the principle of least privilege, then investigate and remediate an intentional access-control misconfiguration.

The project uses PowerShell to manage users, inspect permissions, configure access rules, and verify access through different user accounts.

## Objectives

- Create and manage local Windows user accounts and security groups.
- Organize company files into department-specific directories.
- Configure NTFS permissions using group-based access control.
- Apply the principle of least privilege.
- Test authorized and unauthorized access using different user accounts.
- Introduce an intentional permission misconfiguration.
- Investigate the cause of excessive access.
- Remediate the issue and verify the corrected permissions.

## Lab Environment

**Operating System:** Windows 11 Virtual Machine

**Tools:**
- Windows PowerShell
- Local Users and Groups
- NTFS Access Control Lists (ACLs)
- Windows File System

**User Groups:**
- HR
- IT
- Employees

Six local user accounts were created and assigned to their corresponding department groups.

## Directory Structure

```text
Company/
├── HR/
│   ├── General/
│   └── Confidential/
├── IT/
│   ├── General/
│   └── Restricted/
└── Employees/
    └── Shared/
```

Fictional company documents were created to simulate a realistic shared-drive environment.

## Access Control Policy

The intended access model is based on department responsibilities and least privilege.

| Directory | HR | IT | Employees |
|---|---|---|---|
| HR\General | Modify | No access | No access |
| HR\Confidential | Modify | No access | No access |
| IT\General | No access | Modify | No access |
| IT\Restricted | No access | Modify | No access |
| Employees\Shared | Read | Read | Modify |

*This table represents the intended policy. Not every permission in the table has been implemented or tested yet.*

## Work Completed

- Created six local user accounts and three department security groups.
- Built the company directory structure and populated it with fictional documents.
- Inspected existing NTFS permissions using PowerShell and `Get-Acl`.
- Assigned HR Modify permissions to the HR Confidential directory.
- Verified authorized HR access using an HR account.
- Confirmed that an Employee account was initially denied access to HR Confidential.
- Assigned Employees Modify permissions to the Employees Shared directory.
- Verified that an Employee account could successfully modify and read files in its authorized shared directory.
- Deliberately granted the Employees group Read access to HR Confidential.
- Successfully reproduced unauthorized access using an Employee account.

## Security Findings

An intentional permissions misconfiguration demonstrated how excessive access can expose confidential company information.

After the Employees group was granted explicit Read permission to HR Confidential, an Employee account was able to enumerate the directory
