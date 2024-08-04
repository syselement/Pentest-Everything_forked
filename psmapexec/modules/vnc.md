# VNC

### Description

This module  searches for VNC passwords stored in the registry and configuration files for various VNC implementations, including RealVNC, TightVNC, TigerVNC, and UltraVNC. The module  identifies and decrypts these passwords using the DES algorithm with a fixed key. It covers the following VNC implementations:

* RealVNC: Searches the registry for VNC server proxy credentials.
* TightVNC: Searches the registry for server passwords, control passwords, and view-only passwords.
* TigerVNC: Searches the registry for server passwords.
* UltraVNC: Searches for passwords in configuration files located in specified directories.

For each system output is stored in `$pwd\PME\PME\VNC\`

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
PsMapExec -Username [User] -Password [Pass] -targets [All] -Module VNC -Method [Method] -ShowOutput
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
