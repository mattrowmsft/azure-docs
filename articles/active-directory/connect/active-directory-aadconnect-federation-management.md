---
title: Active Directory Federation Services management and customization with Azure AD Connect | Microsoft Docs
description: AD FS management with Azure AD Connect and customization of user AD FS sign-in experience with Azure AD Connect and PowerShell.
keywords: AD FS, ADFS, AD FS management, AAD Connect, Connect, sign-in, AD FS customization, repair trust, O365, federation, relying party
services: active-directory
documentationcenter: ''
author: anandyadavmsft
manager: femila
editor: ''

ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory    
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 4/4/2016
ms.author: anandy

---
# Manage and customize Active Directory Federation Services by using Azure AD Connect
This article describes how to manage and customize Active Directory Federation Services (AD FS) by using Azure Active Directory (Azure AD) Connect. It also includes other common AD FS tasks that you might need to do for a complete configuration of an AD FS farm.

| Topic | What it covers |
|:--- |:--- |
| **Manage AD FS** | |
| [Repair the trust](#repairthetrust) |How to repair the federation trust with Office 365. |
| [Federate with Azure AD using AlternateID](#alternateid) | Configure federation using alternate ID |
| [Add an AD FS server](#addadfsserver) |How to expand an AD FS farm with an additional AD FS server. |
| [Add an AD FS Web Application Proxy server](#addwapserver) |How to expand an AD FS farm with an additional Web Application Proxy (WAP) server. |
| [Add a federated domain](#addfeddomain) |How to add a federated domain. |
| [Update the SSL certificate](active-directory-aadconnectfed-ssl-update.md)| How to update the SSL certificate for an AD FS farm. |
| **Customize AD FS** | |
| [Add a custom company logo or illustration](#customlogo) |How to customize an AD FS sign-in page with a company logo and illustration. |
| [Add a sign-in description](#addsignindescription) |How to add a sign-in page description. |
| [Modify AD FS claim rules](#modclaims) |How to modify AD FS claims for various federation scenarios. |

## Manage AD FS
You can perform various AD FS-related tasks in Azure AD Connect with minimal user intervention by using the Azure AD Connect wizard. After you've finished installing Azure AD Connect by running the wizard, you can run the wizard again to perform additional tasks.

## Repair the trust <a name=repairthetrust></a>
You can use Azure AD Connect to check the current health of the AD FS and Azure AD trust and take appropriate actions to repair the trust. Follow these steps to repair your Azure AD and AD FS trust.

1. Select **Repair AAD and ADFS Trust** from the list of additional tasks.
   ![Repair AAD and ADFS Trust](media/active-directory-aadconnect-federation-management/RepairADTrust1.PNG)

2. On the **Connect to Azure AD** page, provide your global administrator credentials for Azure AD, and click **Next**.
   ![Connect to Azure AD](media/active-directory-aadconnect-federation-management/RepairADTrust2.PNG)

3. On the **Remote access credentials** page, enter the credentials for the domain administrator.

   ![Remote access credentials](media/active-directory-aadconnect-federation-management/RepairADTrust3.PNG)

    After you click **Next**, Azure AD Connect checks for certificate health and shows any issues.

    ![State of certificates](media/active-directory-aadconnect-federation-management/RepairADTrust4.PNG)

    The **Ready to configure** page shows the list of actions that will be performed to repair the trust.

    ![Ready to configure](media/active-directory-aadconnect-federation-management/RepairADTrust5.PNG)

4. Click **Install** to repair the trust.

> [!NOTE]
> Azure AD Connect can only repair or act on certificates that are self-signed. Azure AD Connect can't repair third-party certificates.

## Federate with Azure AD using AlternateID <a name=alternateid></a>
It is recommended that the on-premises UserPrincipalName and the cloud UserPrincipalName is kept the same. If the on-premises UPN uses a non-routable domain (ex. Contoso.local), or the existing UPN cannot be changed due to local application dependencies, we recommend setting up alternate login ID. Alternate login ID allows you to configure a sign in experience where users can sign in with an attribute other than their UPN, such as mail. Azure AD Connect when providing the choice for UserPrincipalName will default to the userPrincipalName attribute in active directory. While federating using AD FS, if you chose any other attribute for UserPrincipalName, then Azure AD Connect will configure AD FS for alternateID. An example of chosing a different attribute for userPrincipalName is shown below:

![Alternate ID attribute selection](media/active-directory-aadconnect-federation-management/attributeselection.png)

Configuring alternateID for AD FS consists of two main steps:
1. **Configure the right set of issuance claims**: The issuance claim rules for the Azure AD relying party trust needs to be modified to reflect the correct userPrincipalName as per the attribute selected to represent the alternate ID of the user.
2. **Enabling alternateID as part of the AD FS configuration**: AD FS configuration needs to be updated for enabling AD FS to lookup users in the appropriate forests using alternate ID. The configuration is supported for 2012R2 (with [KB2919355](http://go.microsoft.com/fwlink/?LinkID=396590)) or newer. Azure AD Connect makes a check for the presence of the required KB in case the selected servers for AD FS farm are 2012R2. In case the KB is not detected, a warning will be displayed during configuration as shown below:

    ![Warning for missing KB on 2012R2](media/active-directory-aadconnect-federation-management/kbwarning.png)

    To rectify the configuration in case of missing KB, install the required [KB2919355](http://go.microsoft.com/fwlink/?LinkID=396590) and then repair the trust using [Repair AAD and AD FS Trust](#repairthetrust).

> [!NOTE]
> For more information on alternateID and steps to manually configure, read [Configuring Alternate Login ID](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/configuring-alternate-login-id)

## Add an AD FS server <a name=addadfsserver></a>

> [!NOTE]
> To add an AD FS server, Azure AD Connect requires the PFX certificate. Therefore, you can perform this operation only if you configured the AD FS farm by using Azure AD Connect.

1. Select **Deploy an additional Federation Server**, and click **Next**.

   ![Additional federation server](media/active-directory-aadconnect-federation-management/AddNewADFSServer1.PNG)

2. On the **Connect to Azure AD** page, enter your global administrator credentials for Azure AD, and click **Next**.

   ![Connect to Azure AD](media/active-directory-aadconnect-federation-management/AddNewADFSServer2.PNG)

3. Provide the domain administrator credentials.

   ![Domain administrator credentials](media/active-directory-aadconnect-federation-management/AddNewADFSServer3.PNG)

4. Azure AD Connect asks for the password of the PFX file that you provided while configuring your new AD FS farm with Azure AD Connect. Click **Enter Password** to provide the password for the PFX file.

   ![Certificate password](media/active-directory-aadconnect-federation-management/AddNewADFSServer4.PNG)

    ![Specify SSL certificate](media/active-directory-aadconnect-federation-management/AddNewADFSServer5.PNG)

5. On the **AD FS Servers** page, enter the server name or IP address to be added to the AD FS farm.

   ![AD FS servers](media/active-directory-aadconnect-federation-management/AddNewADFSServer6.PNG)

6. Click **Next**, and go through the final **Configure** page. After Azure AD Connect has finished adding the servers to the AD FS farm, you will be given the option to verify the connectivity.

   ![Ready to configure](media/active-directory-aadconnect-federation-management/AddNewADFSServer7.PNG)

    ![Installation complete](media/active-directory-aadconnect-federation-management/AddNewADFSServer8.PNG)

## Add an AD FS WAP server <a name=addwapserver></a>

> [!NOTE]
> To add a WAP server, Azure AD Connect requires the PFX certificate. Therefore, you can only perform this operation if you configured the AD FS farm by using Azure AD Connect.

1. Select **Deploy Web Application Proxy** from the list of available tasks.

   ![Deploy Web Application Proxy](media/active-directory-aadconnect-federation-management/WapServer1.PNG)

2. Provide the Azure global administrator credentials.

   ![Connect to Azure AD](media/active-directory-aadconnect-federation-management/wapserver2.PNG)

3. On the **Specify SSL certificate** page, provide the password for the PFX file that you provided when you configured the AD FS farm with Azure AD Connect.
   ![Certificate password](media/active-directory-aadconnect-federation-management/WapServer3.PNG)

    ![Specify SSL certificate](media/active-directory-aadconnect-federation-management/WapServer4.PNG)

4. Add the server to be added as a WAP server. Because the WAP server might not be joined to the domain, the wizard asks for administrative credentials to the server being added.

   ![Administrative server credentials](media/active-directory-aadconnect-federation-management/WapServer5.PNG)

5. On the **Proxy trust credentials** page, provide administrative credentials to configure the proxy trust and access the primary server in the AD FS farm.

   ![Proxy trust credentials](media/active-directory-aadconnect-federation-management/WapServer6.PNG)

6. On the **Ready to configure** page, the wizard shows the list of actions that will be performed.

   ![Ready to configure](media/active-directory-aadconnect-federation-management/WapServer7.PNG)

7. Click **Install** to finish the configuration. After the configuration is complete, the wizard gives you the option to verify the connectivity to the servers. Click **Verify** to check connectivity.

   ![Installation complete](media/active-directory-aadconnect-federation-management/WapServer8.PNG)

## Add a federated domain <a name=addfeddomain></a>

It's easy to add a domain to be federated with Azure AD by using Azure AD Connect. Azure AD Connect adds the domain for federation and modifies the claim rules to correctly reflect the issuer when you have multiple domains federated with Azure AD.

1. To add a federated domain, select the task **Add an additional Azure AD domain**.

   ![Additional Azure AD domain](media/active-directory-aadconnect-federation-management/AdditionalDomain1.PNG)

2. On the next page of the wizard, provide the global administrator credentials for Azure AD.

   ![Connect to Azure AD](media/active-directory-aadconnect-federation-management/AdditionalDomain2.PNG)

3. On the **Remote access credentials** page, provide the domain administrator credentials.

   ![Remote access credentials](media/active-directory-aadconnect-federation-management/additionaldomain3.PNG)

4. On the next page, the wizard provides a list of Azure AD domains that you can federate your on-premises directory with. Choose the domain from the list.

   ![Azure AD domain](media/active-directory-aadconnect-federation-management/AdditionalDomain4.PNG)

    After you choose the domain, the wizard provides you with appropriate information about further actions that the wizard will take and the impact of the configuration. In some cases, if you select a domain that isn't yet verified in Azure AD, the wizard provides you with information to help you verify the domain. See [Add your custom domain name to Azure Active Directory](../active-directory-add-domain.md) for more details.

5. Click **Next**. The **Ready to configure** page shows the list of actions that Azure AD Connect will perform. Click **Install** to finish the configuration.

   ![Ready to configure](media/active-directory-aadconnect-federation-management/AdditionalDomain5.PNG)

## AD FS customization
The following sections provide details about some of the common tasks that you might have to perform when you customize your AD FS sign-in page.

## Add a custom company logo or illustration <a name=customlogo></a>
To change the logo of the company that's displayed on the **Sign-in** page, use the following Windows PowerShell cmdlet and syntax.

> [!NOTE]
> The recommended dimensions for the logo are 260 x 35 @ 96 dpi with a file size no greater than 10 KB.

    Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.PNG"}

> [!NOTE]
> The *TargetName* parameter is required. The default theme that's released with AD FS is named Default.

## Add a sign-in description <a name=addsignindescription></a>
To add a sign-in page description to the **Sign-in page**, use the following Windows PowerShell cmdlet and syntax.

    Set-AdfsGlobalWebContent -SignInPageDescriptionText "<p>Sign-in to Contoso requires device registration. Click <A href='http://fs1.contoso.com/deviceregistration/'>here</A> for more information.</p>"

## Modify AD FS claim rules <a name=modclaims></a>
AD FS supports a rich claim language that you can use to create custom claim rules. For more information, see [The Role of the Claim Rule Language](https://technet.microsoft.com/library/dd807118.aspx).

The following sections describe how you can write custom rules for some scenarios that relate to Azure AD and AD FS federation.

### Immutable ID conditional on a value being present in the attribute
Azure AD Connect lets you specify an attribute to be used as a source anchor when objects are synced to Azure AD. If the value in the custom attribute is not empty, you might want to issue an immutable ID claim.

For example, you might select **ms-ds-consistencyguid** as the attribute for the source anchor and issue **ImmutableID** as **ms-ds-consistencyguid** in case the attribute has a value against it. If there's no value against the attribute, issue **objectGuid** as the immutable ID. You can construct the set of custom claim rules as described in the following section.

**Rule 1: Query attributes**

    c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
    => add(store = "Active Directory", types = ("http://contoso.com/ws/2016/02/identity/claims/objectguid", "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"), query = "; objectGuid,ms-ds-consistencyguid;{0}", param = c.Value);

In this rule, you're querying the values of **ms-ds-consistencyguid** and **objectGuid** for the user from Active Directory. Change the store name to an appropriate store name in your AD FS deployment. Also change the claims type to a proper claims type for your federation, as defined for **objectGuid** and **ms-ds-consistencyguid**.

Also, by using **add** and not **issue**, you avoid adding an outgoing issue for the entity, and can use the values as intermediate values. You will issue the claim in a later rule after you establish which value to use as the immutable ID.

**Rule 2: Check if ms-ds-consistencyguid exists for the user**

    NOT EXISTS([Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"])
    => add(Type = "urn:anandmsft:tmp/idflag", Value = "useguid");

This rule defines a temporary flag called **idflag** that is set to **useguid** if there's no **ms-ds-consistencyguid** populated for the user. The logic behind this is the fact that AD FS doesn't allow empty claims. So when you add claims http://contoso.com/ws/2016/02/identity/claims/objectguid and http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid in Rule 1, you end up with an **msdsconsistencyguid** claim only if the value is populated for the user. If it isn't populated, AD FS sees that it will have an empty value and drops it immediately. All objects will have **objectGuid**, so that claim will always be there after Rule 1 is executed.

**Rule 3: Issue ms-ds-consistencyguid as immutable ID if it's present**

    c:[Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c.Value);

This is an implicit **Exist** check. If the value for the claim exists, then issue that as the immutable ID. The previous example uses the **nameidentifier** claim. You'll have to change this to the appropriate claim type for the immutable ID in your environment.

**Rule 4: Issue objectGuid as immutable ID if ms-ds-consistencyGuid is not present**

    c1:[Type == "urn:anandmsft:tmp/idflag", Value =~ "useguid"]
    && c2:[Type == "http://contoso.com/ws/2016/02/identity/claims/objectguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c2.Value);

In this rule, you're simply checking the temporary flag **idflag**. You decide whether to issue the claim based on its value.

> [!NOTE]
> The sequence of these rules is important.

### SSO with a subdomain UPN
You can add more than one domain to be federated by using Azure AD Connect, as described in [Add a new federated domain](active-directory-aadconnect-federation-management.md#addfeddomain). You must modify the user principal name (UPN) claim so that the issuer ID corresponds to the root domain and not the subdomain, because the federated root domain also covers the child.

By default, the claim rule for issuer ID is set as:

    c:[Type
    == “http://schemas.xmlsoap.org/claims/UPN“]

    => issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(c.Value, “.+@(?<domain>.+)“, “http://${domain}/adfs/services/trust/“));

![Default issuer ID claim](media/active-directory-aadconnect-federation-management/issuer_id_default.png)

The default rule simply takes the UPN suffix and uses it in the issuer ID claim. For example, John is a user in sub.contoso.com, and contoso.com is federated with Azure AD. John enters john@sub.contoso.com as the username while signing in to Azure AD. The default issuer ID claim rule in AD FS handles it in the following manner:

    c:[Type
    == “http://schemas.xmlsoap.org/claims/UPN“]

    => issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(john@sub.contoso.com, “.+@(?<domain>.+)“, “http://${domain}/adfs/services/trust/“));

**Claim value:**  http://sub.contoso.com/adfs/services/trust/

To have only the root domain in the issuer claim value, change the claim rule to match the following:

    c:[Type == “http://schemas.xmlsoap.org/claims/UPN“]

    => issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(c.Value, “^((.*)([.|@]))?(?<domain>[^.]*[.].*)$”, “http://${domain}/adfs/services/trust/“));

## Next steps
Learn more about [user sign-in options](active-directory-aadconnect-user-signin.md).
