# Day 16: The Wareville’s Key Vault grew three sizes that day.

**Learning Objectives**
- Learn about Azure, what it is and why it is used.
- Learn about Azure services like Azure Key Vault and Microsoft Entra ID.
- Learn how to interact with an Azure tenant using Azure Cloud Shell.


### Intro to Azure
Azure is a CSP (Cloud Service Provider), and CSPs (others include Google Cloud and AWS) provide computing resources such as computing power on demand in a highly scalable fashion. In other words, McSkidy could instead have Azure manage her underlying infrastructure, scaling it in times of increased demand and decreasing it once traffic resumed to normal levels. The best bit? McSkidy only has to pay for what she uses.

Azure (and cloud adoption in general) boasts many benefits beyond cost optimisation. Azure also gave McSkidy access to lots of cloud services. These services can be used to build, deploy, and manage McSkidy's current infrastructure. will come up during the Glitch's attack path. Let's take a look at them now:

**Azure Key Vault**
Azure Key Vault is an Azure service that allows users to securely store and access secrets. These secrets can be anything from API Keys, certificates, passwords, cryptographic keys, and more - essentially, anything you want to keep safe, away from the eyes of others, and easily configure and restrict access to.

The secrets are stored in vaults, which are created by **vault owners**. Vault owners have full access and control over the vault, including the ability to enable auditing so a record is kept of who accessed what secrets and grant permissions for other users to access the vault (known as **vault consumers**). McSkidy uses this service to store secrets related to evidence and has been entrusted to store some of Wareville's town secrets here.

**Microsoft Entra ID**
McSkidy also needed a way to grant users access to her system and be able to secure and organise their access easily. So, a Wareville town member could easily access or update their secret. Microsoft Entra ID (formerly known as Azure Active Directory) is Azure's solution. Entra ID is an identity and access management (IAM) service. It has the information needed to assess whether a user/application can access X resource. In the case of the Wareville town members, they made an Entra ID account, and McSkidy assigned the appropriate permissions to this account.

With that covered, let's see what the Glitch has come up with.

### Assumed Breach Scenario
Knowing that a potential breach had happened, McSkidy decided to conduct an Assumed Breach testing within their Azure tenant. The Assumed Breach scenario is a type of penetration testing setup in which an initial access or foothold is provided, mimicking the scenario in which an attacker has already established its access inside the internal network.

In this setup, the mindset is to assess how far an attacker can go once they get inside your network, including all possible attack paths that could branch out from the defined starting point of intrusion.

#### Azure Cloud Shell
Azure Cloud Shell is a browser-based command-line interface that provides developers and IT professionals a convenient and powerful way to manage Azure resources. It integrates both Bash and PowerShell environments, allowing users to execute scripts, manage Azure services, and run commands directly from their web browser without needing local installation. Cloud Shell has built-in tools and pre-configured environments, including Azure CLI, Azure PowerShell, and popular development tools, making it an efficient solution for cloud management and automation tasks.

#### Azure CLI
Azure Command-Line Interface, or Azure CLI, is a command-line tool for managing and configuring Azure resources. The Glitch relied heavily on this tool while reviewing the Wareville tenant, so let's use the same one while walking through the Azure attack path.

As mentioned, Azure CLI is part of the built-in tools inside the Cloud Shell, so go to the Azure portal and launch Azure Cloud Shell by clicking on the terminal icon.

### Going Down the Azure Rabbit Hole
When the Glitch got hold of an initial account in Wareville's Azure tenant, he had no idea what was inside it. So, he decided to enumerate first the existing users and groups within the tenant.

#### Entra ID Enumeration
1. Using the current account, let's start by listing all the users in the tenant.
	- Execute `az ad user list`
	- Azure CLI typically uses the following command syntax: `az GROUP SUBGROUP ACTION OPTIONAL_PARAMETERS`. Given this, the command above can be broken down into:
		 - Target **group** or service: `ad` (Azure AD or Entra ID)
		 - Target **subgroup**: `user` (Azure AD users)
		 - **Action**: `list`

2. After executing the command, you might have been overwhelmed with the number of accounts listed. For a better view, let's follow McSkidy's suggestion to only look for the accounts prepended with `wvusr-`. According to her, these accounts are more interesting than the other ones. To do this, we will use the `--filter` parameter and filter all accounts that start with `wvusr-`.
	- Execute `az ad user list --filter "startsWith('wvusr-', displayName)"`

3. You may observe that an unusual parameter was set to a specific account in the output. One of the users, **wvusr-backupware**, has its password stored in one of the fields:

![image](https://github.com/user-attachments/assets/9602ff41-26b9-482e-89a1-86b91c0ae839)

4. When the Glitch saw this one, he immediately thought it could be the first step taken by the intruder to gain further access inside the tenant. However, he decided to continue the initial reconnaissance of users and groups. Now, let's continue by listing the groups.
	- Execute `az ad group list`

![image](https://github.com/user-attachments/assets/a296b173-1c50-428c-89de-335688ba98ae)

5. Given the output, it can be seen that a group named `Secret Recovery Group` exists. This is kind of an interesting group because of the description, so let's follow the white rabbit and list the members of this group.
	- Execute `az ad group member list --group "Secret Recovery Group"`

![image](https://github.com/user-attachments/assets/42e4e690-bf8b-49c8-93d0-e93c1ff2fbbd)

6. Given the previous output, it looks like everything makes a little sense now. All of the previous commands seem to point to the `wvusr-backupware` account. Since we have seen a potential set of credentials, let's jump to another user by clearing the current Azure CLI account session and logging in with the **new account**.
	- Execute `az account clear`
	- Execute `az login -u EMAIL -p PASSWORD` (Replace the values with the actual email and password of the newly discovered account).

#### Azure Role Assignments  
Since the `wvusr-backupware` account belongs to an interesting group, the Glitch's first hunch is to see whether sensitive or privileged roles are assigned to the group. And his thought was, "It doesn't make sense to name it like this if it can't do anything, right McSkidy?". But before checking the assigned roles, let's have a quick run-through of Azure Role Assignments.

**Azure Role Assignments** define the resources that each user or group can access. When a new user is created via Entra ID, it cannot access any resource by default due to a lack of role. To grant access, an administrator must assign a **role** to let users view or manage a specific resource. The privilege level configured in a role ranges from read-only to full-control. Additionally, **group members can inherit a role** when assigned to a group.  

1. Let's see if a role is assigned to the Secret Recovery Group. We will be using the `--all` option to list all roles within the Azure subscription, and we will be using the `--assignee` option with the group's ID to render only the ones related to our target group.
	- Execute `az role assignment list --assignee REPLACE_WITH_SECRET_RECOVERY_GROUP_ID --all`.

![image](https://github.com/user-attachments/assets/79aa4096-6f35-480b-96c1-b16dc2aa8179)

The output seems slightly overwhelming, so let's break it down:
- First, it can be seen that there are two entries in the output, which means two roles are assigned to the group.
- Based on the `roleDefinitionName` field, the two roles are `Key Vault Reader` and `Key Vault Secrets User`.
- Both entries have the same scope value, pointing to a Microsoft Key Vault resource, specifically on the `warevillesecrets` vault.

Here's the definition of the roles based on the [Microsoft documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles):

![image](https://github.com/user-attachments/assets/76f32994-1c3a-4e35-b0fc-5af6e68c1835)

After seeing both of these roles, McSkidy immediately realised everything! This configuration allowed the attacker to access the sensitive data they were protecting. Now that she knew this, she asked the Glitch to confirm her assumption.

#### Azure Key Vault
With McSkidy's guidance, the Glitch is now tasked to verify if the current account, **wvusr-backupware**, can access the sensitive data.
1. Execute the following command to list the accessible key vaults: `az keyvault list`

![image](https://github.com/user-attachments/assets/5d036c41-6754-4389-9ad6-8bcb0e889d6c)

The output above confirms the key vault discovered from the role assignments named `warevillesecrets`.

2. Let's see if secrets are stored in this key vault.

![image](https://github.com/user-attachments/assets/bf690c50-ba1c-44a4-960c-f4918181c3ef)

After executing the two previous commands, we confirmed that the **Reader** role allows us to view the key vault metadata, specifically the list of key vaults and secrets.

3. Let's confirm whether the current user can access the contents of the discovered secret with the **Key Vault Secrets User** role. This can be done by executing the following command: `az keyvault secret show --vault-name warevillesecrets --name aoc2024`

![image](https://github.com/user-attachments/assets/a171173b-19a4-49d0-bdbe-edb2da45f42c)

"Bingo!" the Glitch exclaimed as he saw the output above. McSkidy had confirmed her nightmare that a regular user could escalate their way into the secrets of Wareville.

With that, the Glitch had helped McSkidy to find the attack path that had been taken to escalate a user’s privileges.

### Answers
1. What is the password for backupware that was leaked? **R3c0v3r_s3cr3ts!**. (See Entra ID Enumeration, no. 3)
2. What is the group ID of the Secret Recovery Group? **7d96660a-02e1-4112-9515-1762d0cb66b7**. (See Entra ID Enumeration, no. 4).
3. What is the name of the vault secret? **aoc2024**.
4. What are the contents of the secret stored in the vault? **WhereIsMyMind1999**.
