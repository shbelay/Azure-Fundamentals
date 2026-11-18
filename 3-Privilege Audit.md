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

My objective was to understand how command-line tools could support efficient, repeatable access reviews, particularly in enterprise environments containing multiple subscriptions and resource groups. 




### Stage 3: Audit Access Using Azure Resource Graph

I continued the audit using **Azure Resource Graph Explorer** and Kusto Query Language (KQL) to examine resource authorization records across accessible scopes.

My objective was to understand how KQL could be used to identify privileged role assignments, filter results, and perform access reviews efficiently across a large Azure environment. Unlike manually reviewing individual IAM blades, Azure Resource Graph provides a centralized way to analyze supported resource authorization records at scale.

**Evidence:** Azure Resource Graph query and results.

<!-- Insert KQL query and results screenshot here. -->

**Observation:** [Document the actual finding.]

### Stage 4: Review Privileged Identity Management (PIM)

The previous audit methods focused primarily on existing active role assignments. However, reviewing active permissions alone does not necessarily provide a complete picture of privileged access.

I examined Azure resource roles in **Microsoft Entra Privileged Identity Management (PIM)** to understand the distinction between active and eligible assignments.

- **Active assignments:** Permissions currently available to the assigned identity.
- **Eligible assignments:** Permissions that an identity can activate when needed, subject to configured activation requirements.

My objective was to identify privileged access that might not be visible through a traditional active-role review and understand how just-in-time access reduces the need for standing privileges.

**Evidence:** PIM active and eligible Azure resource role assignments.

<!-- Insert PIM screenshots here. -->

**Observation:** [Document the actual finding.]

### Stage 5: Hunt for Excessive Permissions

For the final stage, I applied the knowledge gained from the previous four stages to investigate an overprivileged account. My objective was to identify the account, understand its assigned permissions and scope, and determine why the assignment presented a potential security risk.

With restricted audit access, I focused on identifying and documenting the issue rather than modifying or removing permissions.

**Evidence:** Role assignment details and supporting access control records.

<!-- Insert identified account, role and scope screenshots here. -->

**Observation:** [Document the actual account, permissions and associated risk.]

## 🛠️ What Broke / What Surprised Me

One important takeaway was that a traditional review of Azure IAM role assignments does not necessarily provide a complete picture of privileged access. An identity might not currently have an active privileged role but could be eligible to activate one through PIM. Without examining both assignment types, an auditor could overlook potential access risks.

I also learned that reviewing permissions manually becomes increasingly difficult as an environment grows. Command-line tools and Azure Resource Graph provide more efficient ways to collect and analyze access information across multiple resources.

Another important distinction was that Azure RBAC roles and Microsoft Entra directory roles serve different purposes. Reviewing directory administrators alone would not identify every identity with privileged access to Azure resources.

<!-- Add a real troubleshooting challenge or unexpected result from your lab. -->

## 📊 Findings and Recommendations

### Findings

The following table summarizes the security concerns examined during the audit. Specific lab findings should be documented in the investigation stages above.

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

## 💡 What I Learned

- Developed experience with Azure RBAC auditing methods using the Azure Portal, command-line tools, Azure Resource Graph, and PIM.
- Strengthened my understanding of role assignment scopes, permission inheritance, and least privilege.
- Learned how KQL can support resource-level access audits at scale.
- Developed a better understanding of active and eligible privileged role assignments and the benefits of just-in-time access.
- In future audits, I would combine multiple audit methods and examine both active and eligible permissions before drawing conclusions about an identity's access.

## 🧰 Skills Demonstrated

`Azure RBAC` `IAM` `PIM` `Azure CLI` `Azure PowerShell` `Azure Resource Graph` `KQL` `Access Reviews` `Least Privilege` `Cloud Security` `Security Auditing`
