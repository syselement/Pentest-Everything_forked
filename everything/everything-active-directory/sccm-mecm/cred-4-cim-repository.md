# CRED-4 - CIM Repository

### Description

Dump legacy secrets via CIM repository

The Network Access Account (NAA) is a domain account provisioned on a site server. The NAA account is used by SCCM clients to download software from the distribution point. Otherwise, it serves no other purpose within the configuration.

Regardless of whether a NAA account is configured or not, this method may still provide credential material.

Data stored within WMI classes can still exist with the CIM repository file, even long after the WMI class has been deleted or cleared of data. This file is located at `C:\Windows\System32\wbem\Repository\OBJECTS.DATA`&#x20;

SharpDPAPI and SharpSCCM can be used to decrypted the encrypted data blobs and reveal the underlying credentials.

### Requirements

* Local administrator privileges on an SCCM client

## SharpSCCM

```
SharpSCCM.exe local secrets -m disk
```

<figure><img src="../../../.gitbook/assets/image (2135).png" alt=""><figcaption></figcaption></figure>

### Defensive IDs

* [PREVENT-3: Harden or disable network access accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-3/prevent-3\_description.md)
* [PREVENT-4: Configure Enhanced HTTP](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-4/prevent-4\_description.md)
* [PREVENT-10: Enforce the principle of least privilege for accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-10/prevent-10\_description.md)
* [PREVENT-15: Disable legacy network access accounts in Active Directory](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-15/prevent-15\_description.md)
