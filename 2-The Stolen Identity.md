# The Stolen Identity

Microsoft Azure Fundamentals — Entra ID, Chapter 2 / Module 2.7: Stolen Identity

## Scenario

I investigated an identity compromise involving two linked application registrations in a simulated Azure environment. My objective was to reconstruct the attack chain, identify the permissions that made it possible.

## Environment

- **Platform:** Microsoft Azure / Microsoft Entra ID
- **Training:** [Mad Hat Skool](https://www.skool.com/madhat) — *The Stolen Identity*, Module 2: Azure Fundamentals
- **Identity objects:** Users, app registrations, service principals (enterprise applications), app owners
- **Tools:** Microsoft Entra admin center / Azure portal

## Investigation

### 1 — Entry: Tracing the compromised identity

A user named Carl from accounting was phished with a fake login page. Carl entered his credentials that were stolen and completed MFA requirements that allowed the attacker to steal the authenticated session token. The session token allowed the attacker to bypass Conditional Access policies so the login didn't trigger any alers. Unfortunately, Carl had owner rights on a legacy connector application which provided the attacker with privileged access.

> **Go to App registrations**
<img width="652" height="342" alt="image" src="https://github.com/user-attachments/assets/34c68a40-f67c-4b02-8950-d41be8bcafed" />

> **Go to All applications and click Legacy-Sync-Service**
<img width="484" height="747" alt="image" src="https://github.com/user-attachments/assets/676bf098-98a0-45a7-b539-54e6bf58276d" />

> **Go to Branding & properties**
<img width="975" height="759" alt="image" src="https://github.com/user-attachments/assets/a2ca5693-6770-4071-af04-691d12901e64" />

### Stage 2 — Escalate: Examining the long-lived application secret

Next, I reviewed the legacy app's **Certificates & secrets** blade. The attacker used Carl's owner access to create a client secret. The attacker authenticates as the application's privileged service principal, allowing them to access the directory using the application's existing permissions.

> **Go to Certificates & secrets**
> <img width="975" height="759" alt="image" src="https://github.com/user-attachments/assets/f1a74ffd-82c9-4b11-af17-dacad9564131" />

> **The attacker generated a new secret**
> <img width="1252" height="460" alt="image" src="https://github.com/user-attachments/assets/3f50b0fb-952b-4926-9107-980d35c40164" />


### Stage 3 — Pivot: Identifying a second app as an owner

I returned to the legacy application's **Owners** list and traced a relationship to a second, attacker-controlled application/service principal (Labs-App). The attacker registered this second app (Labs-App) and made its service principal an owner of the legacy app that allowed persistance for the attacker.

> **Reviewed Owners under Legacy-Sync-Service**
> <img width="717" height="607" alt="image" src="https://github.com/user-attachments/assets/9cc07df2-81e0-435a-b84f-dc3705e9d6a9" />

> **Review App registrations. The attacker registred a new app**
<img width="497" height="751" alt="image" src="https://github.com/user-attachments/assets/469c566e-a721-4f25-845c-7e3a48fbbc07" />

> **Go to Branding & properties. Retrieve flag from Internal notes**
<img width="1007" height="780" alt="image" src="https://github.com/user-attachments/assets/20de1a0a-40a3-4c20-8902-6d3b806c1894" />


### Stage 4 — Persist: Reviewing the exposed API scope

I examined **Expose an API** on the Legacy-Sync-Service application and found that there was a custom delegated scope. This new scope makes the Legacy-Sync-Service application addressable as an API resource for clients that request that permission. If the attacker's client secret is removed from the legacy application, they could still leverage the exposed custom API scope and their rogue Labs-App application to launch a separate phishing campaign.

> **Go to Expose an API**
<img width="405" height="513" alt="image" src="https://github.com/user-attachments/assets/818de223-40a3-49e8-83a8-a4423954398a" />

> **New scope added**
<img width="1287" height="389" alt="image" src="https://github.com/user-attachments/assets/e76b64fb-30ea-47d7-b870-2da3009ff86a" />

> **Attacker's app service principal added an owner to legacy app**
<img width="1307" height="428" alt="image" src="https://github.com/user-attachments/assets/8aa7085f-9025-4ba0-b82b-3431567e1823" />

### Stage 5 — Loot

I examined the attacker's Labs-App application authentication configuration and its redirect URI. The redirect URI pointed to the attacker's infrastructure with the intention of stealing victim's credentials and session tokens. 

> **Attacker's redirect URI that looks like a legit login interface**
<img width="1131" height="522" alt="image" src="https://github.com/user-attachments/assets/571ca47c-1187-48f0-90cc-b99ad1fe742f" />


### Investigation summary

| Stage | Where I looked | Security significance |
|---|---|---|
| Entry | Legacy app Owners; initial-access scenario | A compromised user retained unnecessary application ownership |
| Escalate | Certificates & secrets; API permissions | Long-lived app credentials enabled workload identity authentication |
| Pivot | Legacy app Owners; second app registration | Rogue service principal ownership created a way to re-establish credentials |
| Persist | Expose an API | A custom delegated scope enabled an additional OAuth authorization path |
| Loot | Rogue app Authentication; consent configuration | Redirect URI and consent flow could direct authorization artifacts to an attacker-controlled client |


## 📊 Findings and Recommendations

### Findings

1. **Excessive application ownership:** The compromised user retained ownership of a legacy application, creating an avoidable privilege path.
2. **Risky credential lifetime:** A newly created secret had an exceptionally long expiry, increasing the period during which a stolen credential could be abused.
3. **Application-to-application persistence:** A rogue service principal appeared in the legacy application's ownership chain, creating a possible route to re-issue credentials.
4. **Additional OAuth attack surface:** A custom exposed API scope and attacker-controlled redirect URI formed part of the simulated consent-phishing chain.
5. **Containment gap:** Password resets, MFA enforcement, and session revocation alone do not necessarily revoke existing OAuth delegated permission grants or remove app credentials and ownership relationships.

### Recommendations

1. **Contain the application compromise:** Revoke the malicious client secret; remove the rogue service principal from the legacy app's Owners list; remove unauthorized redirect URIs and the custom exposed API scope after validating dependencies. Disable or remove the rogue app as appropriate under incident response procedures.
2. **Revoke persistent authorization:** Identify and explicitly revoke unauthorized **OAuth2PermissionGrant** delegated consent records. Also review and remove unauthorized app-role assignments and credentials where present. Revoke user sessions separately as needed.
3. **Reduce privilege and improve detection:** Review Microsoft Graph application permissions and admin consent; restrict who can register applications when operationally appropriate; regularly audit application Owners; and monitor audit logs for new credentials, owners, consent grants, scopes, and redirect URI changes.
