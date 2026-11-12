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

I examined **Expose an API** on the legacy application and found that the lab's attack path included a custom delegated scope. Publishing a scope makes the application addressable as an API resource for clients that request that permission; it does **not**, by itself, grant consent or establish attacker access.

I then connected that scope to the second application's role in the simulated OAuth flow. The exposed scope provided another way to request authorization involving the trusted legacy application, subject to the tenant's consent and authorization controls.

**Evidence to capture:** **Expose an API** showing the existence of the custom scope. Crop or redact its display name and any embedded challenge text.

![Stage 4 — Exposed API scope](screenshots/04-exposed-api-scope.png)

### Stage 5 — Loot: Reconstructing the consent-phishing path

Finally, I examined the second application's **Authentication** configuration and its redirect URI. In the simulated chain, the redirect URI pointed to attacker-controlled infrastructure. I connected the rogue app's client ID, the custom scope, and the redirect URI to understand how a deceptive OAuth authorization request could send an authorization code to the registered callback endpoint after a user approved consent.

A code alone is not equivalent to a usable token: the application must also satisfy the authorization-code exchange requirements, including any applicable client authentication or PKCE requirements. The key investigative concern was the combination of a deceptive consent request, an attacker-controlled client, and an authorization grant that could survive remediation of the user's password or session.

**Evidence to capture:** Redirect URI configuration with the hostname redacted; API permissions and consent details where visible. Do not publish a functional phishing link.

![Stage 5 — Redirect URI configuration](screenshots/05-redirect-uri.png)

### Investigation summary

| Stage | Where I looked | Security significance |
|---|---|---|
| Entry | Legacy app Owners; initial-access scenario | A compromised user retained unnecessary application ownership |
| Escalate | Certificates & secrets; API permissions | Long-lived app credentials enabled workload identity authentication |
| Pivot | Legacy app Owners; second app registration | Rogue service principal ownership created a way to re-establish credentials |
| Persist | Expose an API | A custom delegated scope enabled an additional OAuth authorization path |
| Loot | Rogue app Authentication; consent configuration | Redirect URI and consent flow could direct authorization artifacts to an attacker-controlled client |

## 🛠️ What Broke / What Surprised Me

The most surprising part was how much control could be hidden outside the usual list of privileged directory roles. I would normally begin an identity investigation by checking sign-ins, MFA, and administrator assignments. This lab showed me that **application ownership, credentials, and OAuth grants** can create serious access paths even when the attacker is not a Global Administrator.

I also learned that app registration permissions are distinct from consent permissions. A user may be able to register an application under the tenant's settings, but that does not automatically authorize it to access every API or obtain tenant-wide admin consent. The precise tenant settings and consent policies matter.

**Personal troubleshooting note to complete:** [Describe an actual point of confusion, a view you initially overlooked, or a hypothesis you changed after checking the app configuration.]

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

## 💡 What I Learned

- **Technical:** App-only client credentials and delegated OAuth consent are different authentication/authorization paths, and both require investigation during an app compromise.
- **Identity security:** Application ownership can be a powerful privilege path that is easy to miss when reviewing only directory administrator roles.
- **Persistence:** Removing a malicious secret is insufficient if the attacker still controls an application owner or retains an unauthorized OAuth grant.
- **Investigation method:** Following the relationships between two app registrations made the sequence clearer than treating each configuration page as an isolated finding.
- **What I'd do differently:** I would build a relationship map of users, service principals, app owners, secrets, permissions, and consent grants at the beginning of the investigation, then verify each link against audit logs where available.

## 📸 Evidence and Screenshots

The `screenshots/` paths above are placeholders. Replace them with sanitized screenshots from the actual lab before publishing. Exclude challenge flags, live secrets, access tokens, tenant-specific sensitive identifiers, and functional phishing URLs.

## ⚖️ Scope and Limitations

This report documents a **simulated training scenario** and the investigation approach taught in the lab. The sequence is reconstructed from the scenario and Entra configuration; it is not a claim that I observed a real-world compromise or personally executed the attack. Where actual audit logs, sign-in records, or OAuth grant objects were not reviewed, the report does not claim they were independently verified.
