# WinSCP

This module iterates through  the registry and identifies WinSCP session information, attempts to decrypt it and shows the plaintext session information.

For each system output is stored in `$pwd\PME\PME\WinSCP\`

### **Supported Methods**

* MSSQL&#x20;
* SMB&#x20;
* SessionHunter (WMI)
* WMI&#x20;
* WinRM

### Optional Parameters

<table><thead><tr><th width="159">Parameter</th><th width="99.33333333333331">Value</th><th>Description</th></tr></thead><tbody><tr><td>-ShowOutput</td><td>N/A</td><td>Displays each target output to the console</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Display only successful results</td></tr></tbody></table>

### Usage

{% code overflow="wrap" %}
```powershell
# Standard execution
PsMapExec -Username [User] -Password [Pass] -targets [All] -Module WinSCP -Method [Method] -ShowOutput
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2163).png" alt=""><figcaption></figcaption></figure>
