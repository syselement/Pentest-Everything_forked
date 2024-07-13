# Wi-Fi

Identifies Wi-Fi connection credentials on the target

For each system output is stored in `$pwd\PME\PME\Wi-Fi\`

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
PsMapExec -Username [User] -Password [Pass] -targets [All] -Module WiFi -Method [Method] -ShowOutput
```
{% endcode %}
