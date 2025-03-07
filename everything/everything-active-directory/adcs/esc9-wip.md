# ESC9 - WIP

{% hint style="warning" %}
Document is work in progess and incomplete
{% endhint %}

## Description <a href="#description" id="description"></a>

If the `msPKI-Enrollment-Flag` attribute of a certificate template includes the `CT_FLAG_NO_SECURITY_EXTENSION` value, the `szOID_NTDS_CA_SECURITY_EXT` extension will **not** be included in the issued certificates. As a result, regardless of how the `StrongCertificateBindingEnforcement` registry key is configured—even if it is set to its default value of 1—the certificate mapping process will behave as though the registry key is set to 0. This effectively **bypasses strong certificate mapping**, potentially weakening the security of certificate-based authentication.

What this means is that if we possess access to an account that has Write privileges to a user account, we can modify the target users User Principal Name (UPN), change it to that of another account we wish to compromise.\
\
**Attack Overview**

1. **Modify UPN of Target Account (B):**
   * If an attacker (Account A) has **GenericWrite** permissions on Account B, they can modify Account B's UPN to match that of a higher-privileged account (Account C).
2. **Request Certificate for Account B:**
   * The attacker then requests a certificate for Account B. Due to the modified UPN, the certificate will be issued with the identity of Account C.
3. **Authenticate as Account C:**
   * With the obtained certificate, the attacker can authenticate as Account C, effectively escalating their privileges.

This is not something generally configured within the template options itself but rather by an administrator with certutil.

```
certutil -dstemplate ESC9 msPKI-Enrollment-Flag +0x00080000
```

```
Old Value:
  msPKI-Enrollment-Flag REG_DWORD = 829 (2089)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ALLOW_ENROLL_ON_BEHALF_OF -- 800 (2048)

New Value:
  msPKI-Enrollment-Flag REG_DWORD = 80829 (526377)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ALLOW_ENROLL_ON_BEHALF_OF -- 800 (2048)
    0x80000 (524288)

CertUtil: -dsTemplate command completed successfully.
```

### Requirements for attack path <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

* The **StrongCertificateBindingEnforcement** registry key should remain at its default value of 1 and should not be set to 2, or the CertificateMappingMethods must include the UPN flag (0x4).
* The certificate template must have the `CT_FLAG_NO_SECURITY_EXTENSION` flag included in the `msPKI-Enrollment-Flag` attribute.
* Additionally, the certificate template must explicitly allow **Client Authentication** as one of its intended purposes.
* For privilege escalation, an attacker must have at least GenericWrite permissions on a user account (Account A) to compromise another user account (Account B).

