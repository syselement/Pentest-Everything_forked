# SessionExec

The SessionExec module is based on [Leo4j's SessionExec](https://github.com/Leo4j/SessionExec), it uses a PowerShell port of the code  [Invoke-SessionExec](https://github.com/The-Viper-One/Invoke-SessionExec).

This module will connect to the target system elevate to SYSETM and run a specified `-command` as each user on the system that exhibits a logon session.

For example, assuming the below output. We can see the remote host currently has the users standarduser and srv2019-admin within existing logon sessions. PsMapExec will execute a given command within each user context.

```
C:\Users\SRV2019-Admin>quser
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
 standarduser                              1  Disc            7  04/08/2024 17:14
 srv2019-admin         console             2  Active      none   04/08/2024 17:18
```

Output for  SessionExec is stored `$PWD\PME\SCCM\`.

### **Supported Methods**

* SMB&#x20;
* SessionHunter (WMI)
* WMI&#x20;
* WinRM

### Optional Parameters <a href="#optional-parameters" id="optional-parameters"></a>

<table><thead><tr><th width="185">Parameter</th><th width="124">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Command</td><td>Command</td><td>The command to run as each user, if not specified a simple "whoami" will be executed.</td></tr><tr><td>-ShowOutput</td><td>N/A</td><td>Displays each targets output to the console</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Display only successful results</td></tr></tbody></table>

## Usage

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

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
