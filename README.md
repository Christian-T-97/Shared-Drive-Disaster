# Shared-Drive-Disaster

# Shared Drive Disaster: Windows Access Control & Troubleshooting

## Project Overview

This project simulates a company file-access incident in which employees may have access to confidential documents they are not authorized to view.

The objective is to configure and investigate Windows file permissions, identify access-control problems, apply the principle of least privilege, and verify whether the resulting permissions behave as intended.

The lab uses local Windows user accounts, security groups, NTFS permissions, and PowerShell to explore access control and troubleshooting.

## Objectives

- Create and manage local Windows users and groups.
- Organize company files into departmental directories.
- Configure folder permissions using security groups.
- Apply the principle of least privilege.
- Test authorized and unauthorized file access.
- Investigate permission errors and inherited permissions.
- Document configuration, testing, findings, and remediation.

## Lab Environment

- **Operating system:** Windows 11 virtual machine
- **Administration:** PowerShell
- **Access control:** Windows NTFS permissions and access control lists (ACLs)
- **Accounts:** Local HR, IT, and Employees user groups
- **File structure:** Fictional company documents organized by department and access level

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

## Intended Access Policy

| Directory | HR | IT | Employees |
|---|---|---|---|
| HR/General | Modify | No access by default | No access by default |
| HR/Confidential | Modify | No access by default | No access |
| IT/General | No access by default | Modify | No access by default |
| IT/Restricted | No access by default | Modify | No access |
| Employees/Shared | Read | Read | Modify |

These are the intended permissions. Actual effective access must be tested, including inherited permissions and access granted through other group memberships.

## Progress and Findings

### Completed

- Created local user accounts for HR, IT, and Employees.
- Created departmental security groups and assigned users to their corresponding groups.
- Created the company directory structure and fictional departmental documents.
- Inspected existing folder ACLs using PowerShell and identified inherited permissions.
- Added an explicit `Modify` permission for the HR group on the `HR/Confidential` directory.
- Confirmed that the HR group permission appeared in the ACL output as an explicit, allowed rule.
- Began testing access using an HR account.

### Preliminary HR Access Test

An HR user's credentials were used to launch a separate PowerShell process. The `whoami` command identified the account as `Gervonta`.

A fictional confidential HR text file was opened, edited, and saved successfully while testing the HR user's access.

**Verification note:** The Windows desktop session remained signed in as `Admin1`. The file operation has not yet been independently confirmed to have occurred under Gervonta's security context, so this result remains preliminary.

### Remaining Work

- Confirm HR file access under the HR account's actual security context.
- Test access to confidential HR files using an ordinary employee account.
- Investigate any unexpected access or permission-denied results.
- Correct misconfigured permissions where necessary.
- Verify access after remediation.
- Capture evidence and document final findings.

## Evidence

Screenshots and supporting evidence are stored in the `screenshots/` directory.
![HR confidential file access](screeshots/02-HR-confidential-file-access.png)

## Key Learning

Configuring a permission does not, by itself, prove that access is secure. Effective access depends on explicit permissions, inherited ACLs, group memberships, and the security context of the account performing the operation.

The goal is to verify access through testing and evidence rather than assume that the configuration works as intended.
