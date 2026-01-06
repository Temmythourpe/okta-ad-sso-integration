# Okta & Active Directory SSO Integration Lab

<img width="1127" height="720" alt="image" src="https://github.com/user-attachments/assets/4d390ba3-dd45-43af-8d72-191b769effcc" />

## Project Overview
This project demonstrates a modern **hybrid identity management** setup by integrating an on-premise **Active Directory** environment with **Okta** as a cloud Identity Provider (IdP). 
The goal was to synchronize user accounts and configure **SAML-based Single Sign-On (SSO)** for a cloud application (Slack), showcasing an end-to-end enterprise identity workflow.

**Key Skills Demonstrated:** Identity & Access Management (IAM), Okta Administration, Active Directory Synchronization, SAML/SSO Configuration, Troubleshooting.

## Reference: The Foundation
This project extends the user lifecycle automation built in a previous lab: **[Active Directory User Management](https://github.com/Temmythourpe/Active-Directory-User-Management)**. 
The project automated user onboarding/offboarding in a Windows Server Active Directory environment (`mylab.com`).

## Lab Architecture
This project connected two primary environments:
1.  **On-Premise Lab (mylab.com):** A Windows Server 2022 domain controller hosting Active Directory Domain Services, with users and groups managed by the [(https://github.com/Temmythourpe/Active-Directory-User-Management/blob/main/UserManagement-Project/Scripts/New-UserOnboarding.ps1)].
2.  **Cloud Identity Platform (Okta):** A free Okta Developer account used as the central Identity Provider (IdP).

<img width="220" height="476" alt="Okta-flow-diagram drawio" src="https://github.com/user-attachments/assets/a7be1c1d-951c-4ac4-a7ea-fb7e039e98ce" />

## Implementation Steps

### Step 1: Establish Directory Synchronization with Okta AD Agent
The first step was to create a secure bridge between the on-premise Active Directory and the Okta cloud.

**Actions Taken:**
1. Downloaded the **Okta Active Directory Agent** installer from the Okta Admin Console.
2. Installed the agent on the Windows Server domain controller (`mylab.com`), running it as an administrator.

<img width="498" height="384" alt="Okta-installation" src="https://github.com/user-attachments/assets/982e3e63-b292-4a9c-855e-a46c5affaf5d" />

3.  During installation, configure the agent with the following key settings:
    - **Active Directory Domain:** `mylab.com`
    - **Use SSL Connection:** *Unchecked* (Not required for lab LDAP sync)
    - **Use Proxy Server:** *Unchecked* (Automatic detection for homelab)
    - **Service Account:** Used the default `OktaServiceAccount` created by the installer.
    - **Okta Organization URL:** `https://trial-3229378.okta.com`
4.  Authorized the agent by signing into the Okta Admin account when prompted.

**Outcome:** The agent established a continuous, secure connection, enabling user and group synchronization from AD to Okta.

<img width="947" height="738" alt="image" src="https://github.com/user-attachments/assets/2c2ee986-8f2f-41df-91f7-d7b022609bf0" />
<img width="761" height="483" alt="image" src="https://github.com/user-attachments/assets/cbda205e-b4c5-498a-987e-a03445c3ac2e" />

### Step 2: Configure User Import & Attribute Mapping
With the agent connected, the next task was to define which users to import and how their AD attributes map to Okta's user profile.

<img width="946" height="811" alt="image" src="https://github.com/user-attachments/assets/218eaf20-7c88-4e30-96d3-f7f0c9cbcd65" />

**Actions Taken:**
1.  In the Okta Admin Console, navigated to the `mylab.com` directory settings and selected the **"Import"** tab.
2.  **Set Import Scope:** Selected specific Active Directory security groups (`GRP_IT`, `GRP_HR`) created in the foundational project for synchronization. 
3.  Initiated a **full user import** from the selected AD groups.

<img width="931" height="784" alt="image" src="https://github.com/user-attachments/assets/3679b1e1-8cce-40c4-bf1a-a3538dc256fd" />

**Outcome:** Active Directory user accounts, such as those in the `GRP_IT` and `GRP_HR` groups, were successfully discovered and staged for import into Okta. This step highlighted the importance of attribute mapping in identity synchronization.

### Step 3: Import & Activate Users
The final step of the synchronization workflow was to review and confirm the user import from Active Directory into Okta.

<img width="782" height="697" alt="image" src="https://github.com/user-attachments/assets/48944a52-ea39-461f-b729-7ee3dbbe5d0b" />

**Actions Taken:**
1.  On the **import preview screen**, Okta listed all discovered AD users with the status **"NO Okta user matches found"** and **"NEW Okta user"**, correctly identifying them for initial creation.
2.  All users were selected from the list, and the assignment was confirmed.
3.  The option **"Auto-activate users after confirmation"** was checked to ensure imported accounts were immediately usable.
4.  Finalized the process by clicking **"Confirm Assignments"**.

**Outcome:** 
- User accounts (e.g., `john.smith`, `alice.johnson`) were successfully created in Okta with their source listed as `mylab.com`.
- Users appeared in **Directory > People** with an **Active** status, completing the hybrid identity lifecycle from on-premise AD to the cloud.

<img width="805" height="723" alt="image" src="https://github.com/user-attachments/assets/cba348fc-952c-4b87-a8a5-d233d38c2a30" />

### Step 4: Configure SAML SSO for a Cloud Application (Slack)
The goal was to use Okta as the Identity Provider (IdP) to enable Single Sign-On for a cloud application, demonstrating centralized access control.

**Actions Taken in Okta:**
1.  Added the **Slack** application from the Okta Application Catalog.
2.  Configured the application with the **Slack workspace domain** (`mylab-workspacegroup`).
3.  On the **"Sign On"** tab, configured the **SAML 2.0** settings:
    - **Single sign-on URL:** `https://mylab-workspacegroup.slack.com/sso/saml`
    - **Audience URI (SP Entity ID):** `https://mylab-workspacegroup.slack.com`
    - **Application username format:** Set to `Email` to use the user's email as the identifier.
4.  Saved the configuration, which generated the critical **Identity Provider** metadata:
    - **Identity Provider Single Sign-On URL**
    - **Identity Provider Issuer**
    - **X.509 Certificate**

<img width="920" height="770" alt="image" src="https://github.com/user-attachments/assets/8721c94d-fa6c-422c-896f-0f4f6ba81116" />

**Encountered Real-World Constraint:**
To complete the trust, the IdP metadata from Okta must be entered into Slack's administration console under **Settings & Administration > Authentication**. However, this **SAML SSO feature is exclusive to Slack's paid Business+ or Enterprise Grid plans**, as shown in the admin interface below.

<img width="934" height="503" alt="image" src="https://github.com/user-attachments/assets/8e2854a2-9805-4470-963a-b12cc22a1ac5" />

**Outcome & Professional Insight:**
The SAML configuration on the Okta (IdP) side is fully operational. This project successfully demonstrates the **standard handoff and configuration process** that an Identity Administrator performs. The final activation step depends on the application's (Slack's) licensing model, a common scenario in enterprise IT that separates technical implementation from procurement decisions.
