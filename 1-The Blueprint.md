# Operation Dead Deploy

Microsoft Azure Fundamentals — *The Blueprint*, Chapter 1 / Module 1.6: Operation Dead Deploy  

## Scenario

A junior intern at Mad Hat Labs was temporarily given Contributor access to deploy a test environment in Microsoft Azure. The intern was unfamiliar with the organization's cloud governance standards and provisioned resources without following the requirements and left for the weekend without addressing potential misconfigurations.

As the on-call Azure engineer, I was tasked with investigating the environment using Reader (read-only) access to avoid modifying existing resources. My objective was to identify what had been deployed, assess potential security and governance risks, determine which controls may have failed, and document my findings and recommendations using evidence collected from the Azure Portal.

## Environment

- **Platform:** Microsoft Azure
- **Training:** [Mad Hat Skool](https://www.skool.com/madhat) — *The Blueprint*, Module 1: Azure Fundamentals
- **Services and concepts:** Azure Resource Manager, management groups, subscriptions, resource groups, Azure RBAC, Azure Policy, resource locks, tags, ARM templates, and Bicep

### 1. Identify the Non-Compliant Resource Group

I navigated the subscription in Azure portal and reviewed the existing resource groups for naming inconsistencies. The followed Microsoft naming convention is to include rg- prefix followed by workload identifier, environment, region, and instance number. The objective was to identify any resource group that did not follow the standards.

> **Navigating to resource group** 
<img width="281" height="526" alt="image" src="https://github.com/user-attachments/assets/cbf0bc0c-2234-48d9-8cdc-ee7ac348f7be" />

> **Reviewing resource groups for naming convention violations and the last resource group doesn't follow the standard because it doesn't start with rg-** 
<img width="1637" height="717" alt="image" src="https://github.com/user-attachments/assets/2b4a0e45-dacf-4a68-b6fe-477437b31af0" />

### 2. Inspect the Resource

After identifying the resource group that violated the naming convention, I opened it to view the resources deployed by the intern. The resource group had only one resource. My objective was to identify the intern-flag tag.

> **Clicked the resource group** 
<img width="497" height="462" alt="image" src="https://github.com/user-attachments/assets/a7e0b4cb-36ec-455a-b916-4a9bc65fd58f" />

> **Clicked Tags** 
<img width="956" height="508" alt="image" src="https://github.com/user-attachments/assets/e44aeaa3-421c-4af6-9e0e-0f11e2bf6cc5" />

> **Retrieved intern-flag Tags**
<img width="1867" height="477" alt="image" src="https://github.com/user-attachments/assets/ad279320-4397-4dc5-97a4-f66a4dbf6c87" />


### 3. Trace the Resource Deployment

After examining the resource and its tags, I continued my investigation by reviewing the Azure Resource Manager (ARM) deployment history to determine how and when the resource was provisioned. My objective was to identify the specific deployment responsible for creating the intern's resource.

> **Click Deployments**
<img width="1552" height="635" alt="image" src="https://github.com/user-attachments/assets/258f8f4d-bf65-4213-9fb9-af54f9f69b26" />

> **Deployment Record**
<img width="975" height="374" alt="image" src="https://github.com/user-attachments/assets/9222aaf9-6ec4-43af-a6ba-f916066e5105" />

### 4. Review naming conventions and tags

After confirming the deployment, I investigated why the intern was able to deploy resources that violated the organization's naming conventions with an active Azure Policy assignment. I navigated to the resource group's Policies section and reviewed the compliance status, which showed that the naming convention policy was flagged as Non-compliant.

I discovered that the policy's Effect parameter was set to Audit rather than Deny. This explained why the deployment was allowed to proceed. The Audit effect identifies and reports non-compliant resources but does not block their creation, whereas Deny prevents deployments that violate the policy.

> **Click Policies**
<img width="324" height="707" alt="image" src="https://github.com/user-attachments/assets/a4d0ccdb-0af7-428d-aabf-2d07c8486ffc" />

> **Naming Convention is Non-compliant**
<img width="1862" height="706" alt="image" src="https://github.com/user-attachments/assets/fa792c73-9a38-48cf-9e4a-439f8da2a6b3" />

> **Effect Type set to Audit**
<img width="1473" height="587" alt="image" src="https://github.com/user-attachments/assets/265a572e-787b-4a77-bfd9-3292a8b42b7c" />


### Recommendations

1. **Verify effective access:** Check the deployment identity's permissions at the affected scope and apply least privilege.
2. **Review policy assignments:** Inspect inherited Azure Policy assignments and denial details before requesting an exception or changing configuration.
3. **Maintain governance controls:** Use consistent naming, ownership tags, and appropriate resource locks to support accountability and prevent accidental changes.

## What I Learned

- **Technical:** How management groups, subscriptions, resource groups, and resources relate to access and governance scope.
- **Security:** Why Azure RBAC, Azure Policy, and resource locks must be evaluated separately.
- **Operations:** How naming conventions and tags improve resource identification, ownership, and cost visibility.
- **Investigation:** Why collecting the exact deployment error is more effective than guessing at the cause.
- **What I'd do differently:** Start with the failed operation, its scope, and its error details; then systematically test RBAC, policy, and lock hypotheses.
