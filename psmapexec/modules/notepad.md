# Notepad

This module searched for stored data in Notepad and Notepad++ in the following locations:

*   **Notepad++ backup files**:

    `C:\Users\<UserProfile>\APPDATA\Roaming\NotePad++\backup\`
*   **Microsoft Notepad backup files (Windows 11 / Server 2025 only)**

    `C:\Users\<UserProfile>\AppData\Local\Packages\Microsoft.WindowsNotepad_*\LocalState\TabState\`

{% hint style="info" %}
Default behaviour in Windows 11 and Windows Server 2025 is to store Notepad files on disk in binary files. This module will attempt to extract readable strings from these files.
{% endhint %}

For each system output is stored in `$pwd\PME\PME\Notepad\`

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
PsMapExec -Username [User] -Password [Pass] -targets [All] -Module Notepad -Method [Method] -ShowOutput
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2164).png" alt=""><figcaption></figcaption></figure>
