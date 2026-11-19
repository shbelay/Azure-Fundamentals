# Privilege Audit

Microsoft Azure Fundamentals — Azure RBAC and Privileged Access, Chapter 3 / Module 3.5: Privilege Audit

## Scenario

Following a recent security incident at Mad Hat Labs, the management board requested an audit of access permissions across its Azure environment to understand who had access to resources and whether those privileges were appropriate.

Acting as an IAM security auditor, I was tasked with reviewing Azure role assignments, identifying potentially excessive permissions, and examining active and eligible privileged access. My objective was to gather evidence and document findings and recommendations.

## Environment

- **Platform:** Microsoft Azure
- **Training:** [Mad Hat Skool](https://www.skool.com/madhat)
- **Services:** Azure RBAC, Azure Resource Graph, Privileged Identity Management (PIM)
- **Tools:** Azure Portal, Azure CLI, Azure PowerShell

### 1: Review Access Control (IAM)

I began by navigating to the Mad Hat Labs subscription in the Azure Portal and reviewing the **Access control (IAM)** section. I examined role assignments to understand which identities had access, what roles they had been assigned, and at what scope those permissions applied.

My objective was to identify potentially excessive privileges and determine whether permissions were directly assigned or inherited from a higher scope. I downloaded a CSV role assignment report to examine access permissions.

> **Navigated to Mad Hat Labs subscription**
<img width="1072" height="312" alt="image" src="https://github.com/user-attachments/assets/cd66bc47-dc6f-4ca5-9e6a-f2418163cee3" />

> **Clicked Access Control (IAM)**
<img width="276" height="440" alt="image" src="https://github.com/user-attachments/assets/9c626525-5bff-49c4-85a3-b3f99da08cbf" />

> **Downloaded role assignments**
<img width="1035" height="157" alt="image" src="https://github.com/user-attachments/assets/65d6d4b7-347f-4703-8c90-fe67fbcaeb1b" />

> **A security principal was assigned Owner role to several resources**
<img width="1252" height="446" alt="image" src="https://github.com/user-attachments/assets/3e253b0a-a651-41b5-86d9-0da860d79c42" />


### Stage 2: Audit Role Assignments Using the Command Line

After reviewing role assignments, I explored how Azure CLI and PowerShell could retrieve role assignment information rather than manually navigating through individual resources.

> **Click Azure CLI**
<img width="1172" height="72" alt="image" src="https://github.com/user-attachments/assets/b4b2a8f7-bdc6-4f8c-b3a1-36a09d2b9ab6" />

> **Retreived role assignments through PowerShell**
<img width="712" height="100" alt="image" src="https://github.com/user-attachments/assets/4cc2dfd6-99e7-4411-b365-0f7d1d236195" />

> **Located an orphaned account with active permission**
<img width="701" height="442" alt="image" src="https://github.com/user-attachments/assets/a6d35b75-7167-4dd0-8937-7f47c12c16c8" />



### Stage 3: Audit Access Using Azure Resource Graph

I continued the audit using **Azure Resource Graph Explorer** and Kusto Query Language (KQL) to examine resource authorization records. My objective was to understand how KQL could be used to identify privileged role assignments, filter results, and perform access reviews. 

> **Navigated to Resource Graph**
<img width="431" height="481" alt="image" src="https://github.com/user-attachments/assets/30c31fa5-c7a7-421d-bf28-f75aefb094c8" />

> **Queried authorizationresources table for the orphaned account**
<img width="792" height="162" alt="image" src="https://github.com/user-attachments/assets/c70dbbf0-2594-491f-8f9f-b69ed58b529e" />


### Stage 4: Review Privileged Identity Management (PIM)

I examined Azure resource roles in **Microsoft Entra Privileged Identity Management (PIM)** to understand the distinction between active and eligible assignments.

- **Active assignments:** Permissions currently available to the assigned identity.
- **Eligible assignments:** Permissions that an identity can activate when needed.

> **Navigate to Privileged Identity Management (PIM)**
<img width="522" height="272" alt="image" src="https://github.com/user-attachments/assets/ad6b28c8-f379-4796-bd14-5dae37c12132" />

> **Azure Resources**
<img width="265" height="365" alt="image" src="https://github.com/user-attachments/assets/eb0a46a2-60d2-4450-b090-682ad60b4185" />

> **Click Manage resource after selecting the appropriate drop-down information**
<img width="565" height="587" alt="image" src="https://github.com/user-attachments/assets/811a6c0a-273b-4d25-b77b-ce47fc644882" />

> **Click Assignments**
<img width="271" height="242" alt="image" src="https://github.com/user-attachments/assets/39bcf3dc-4d88-4623-9fbd-b6fc49218d19" />

> **Click Export**
<img width="1046" height="78" alt="image" src="https://github.com/user-attachments/assets/460c12cc-0701-46f4-9d93-8a50ea529ada" />

> **Click Start**
<img width="354" height="865" alt="image" src="https://github.com/user-attachments/assets/614c1da3-1a39-4060-ab00-91655033a66b" />

> **Review audit report**
<img width="1867" height="627" alt="image" src="https://github.com/user-attachments/assets/d6402ed6-2a5e-4541-a3e1-071393c989be" />


### Stage 5: Hunt for Excessive Permissions

For the final stage, I applied the knowledge gained from the previous four stages to investigate an overprivileged account. My objective was to identify the account, understand its assigned permissions and scope, and determine why the assignment presented a potential security risk.

> **Navigated to PIM and clicked Activate**
> <img width="1025" height="537" alt="image" src="https://github.com/user-attachments/assets/ff481a58-c159-42a1-bbcb-db0023d1edc4" />

> **Activate for operative role**
<img width="1475" height="187" alt="image" src="https://github.com/user-attachments/assets/3307bf20-4cac-4145-839b-25153bfda19e" />

> **Resource Group**
<img width="236" height="312" alt="image" src="https://github.com/user-attachments/assets/ad1ebae7-d9f2-4923-9bfc-41ded6481a85" />

> **Downloaded resource assignments and found other identities assigned Owner role to the resource group**
<img width="1713" height="646" alt="image" src="https://github.com/user-attachments/assets/63eda66c-d0e8-4512-a419-a8ade1b31247" />

## Findings and Recommendations

### Findings

| Audit Area | Security Concern |
| --- | --- |
| Azure RBAC | Excessive or unnecessary role assignments |
| Permission inheritance | Privileges inherited from higher scopes |
| Standing access | Privileged permissions that remain continuously active |
| PIM eligibility | Potential privileged access not captured by active-only reviews |
| Overprivileged identities | Permissions exceeding operational requirements |

### Recommendations

1. **Enforce least privilege:** Review existing role assignments and reduce unnecessary Owner, Contributor, and other privileged permissions. Assign roles at the lowest appropriate scope.
2. **Implement just-in-time access:** Use PIM eligible and time-limited assignments where appropriate instead of permanent administrative access.
3. **Conduct recurring access reviews:** Review permissions granted to users, groups, service principals, and managed identities.
4. **Automate access auditing:** Use Azure CLI, Azure PowerShell, and Azure Resource Graph to support repeatable access reviews.
5. **Monitor privileged role changes:** Configure monitoring and alerts for sensitive role assignments and changes to privileged access.

## What I Learned

- Developed experience with Azure RBAC auditing methods using the Azure Portal, command-line tools, Azure Resource Graph, and PIM.
- Strengthened my understanding of role assignment scopes, permission inheritance, and least privilege.
- Learned how KQL can support resource-level access audits at scale.
- Developed a better understanding of active and eligible privileged role assignments and the benefits of just-in-time access.
- In future audits, I would combine multiple audit methods and examine both active and eligible permissions before drawing conclusions about an identity's access.

## Skills Demonstrated

`Azure RBAC` `IAM` `PIM` `Azure CLI` `Azure PowerShell` `Azure Resource Graph` `KQL` `Access Reviews` `Least Privilege` `Cloud Security` `Security Auditing`
