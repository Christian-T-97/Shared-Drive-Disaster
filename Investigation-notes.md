# Shared Drive Disaster — Investigation Notes

## 1. Environment Setup
- Operating system: Windows 11 VM
- Created six local user accounts across HR, IT, and Employees.
- Created three local security groups: HR, IT, and Employees.
- Added users to their corresponding groups.
- Created the Company directory and department subdirectories.
- Added fictional text files to the directories.

## 2. Initial Permission Inspection
- Used PowerShell `Get-Acl` to inspect directory permissions.
- Initial entries included SYSTEM, Administrators, and Admin1.
- These entries showed FullControl and were inherited from parent directories.

## 3. Permission Configuration
- Added an explicit Modify permission for the HR group on the HR\Confidential directory.
- Verified the new ACL entry using PowerShell.

## 4. Access Testing
- HR account access: Pending verification.
- Regular employee access: Pending verification.

## 5. Misconfiguration and Remediation
- To be completed.

## 6. Final Verification
- To be completed.

## 7. Lessons Learned
- To be completed.
