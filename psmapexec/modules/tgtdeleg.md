# TGTDeleg

This module builds upon the SessionExec module. Whereby, execution on a remote host will perform a TGTDeleg operation from Rubeus under each user logon on the remote system.

For example, assuming the below output. We can see the remote host currently has the users standarduser and srv2019-admin within existing logon sessions. PsMapExec will perform Rubeus' TGTDeleg command as each user and obtain a usable TGT.

```
C:\Users\SRV2019-Admin>quser
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
 standarduser                              1  Disc            7  04/08/2024 17:14
 srv2019-admin         console             2  Active      none   04/08/2024 17:18
```

Output for  TGTDeleg is stored `$PWD\PME\TGTDeleg\`.

{% hint style="info" %}
There are some limitations with this module. It is not possible to use TGTDeleg to obtain a useable TGT for a user if they are a member of the "Protected Users" group of if they have the flag "This account is sensitive and cant be delegated" enabled.
{% endhint %}

### Optional Parameters <a href="#optional-parameters" id="optional-parameters"></a>

<table><thead><tr><th width="186">Parameter</th><th width="113">Value</th><th>Description</th></tr></thead><tbody><tr><td>-NoParse</td><td>N/A</td><td>If specified, PsMapexec will not parse the ticket output.</td></tr><tr><td>-ShowOutput</td><td>N/A</td><td>Displays each targets output to the console</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Display only successful results</td></tr></tbody></table>

## Usage

{% code overflow="wrap" %}
```powershell
# Standard execution
PsMapExec -Username [User] -Password [Pass] -targets [All] -Module TGTDeleg -Method [Method]
```
{% endcode %}

### Parsing

PsMapExec will parse the results from each system and present the results in a digestible and readable format. The notes field will highlight in yellow any interesting information about each result.  Additionally, the output will generate easy one liner commands to run to impersonate the user.

The table below shows the possible values for the notes field.

<table><thead><tr><th width="354">Value</th><th>Description</th></tr></thead><tbody><tr><td>AdminCount=1</td><td>Identifies an account that may hold privileged permissions within the domain</td></tr><tr><td>Domain Admin, Enterprise Admin, Server Operator, Account Operator</td><td>The account is a member of one of these privileged groups</td></tr></tbody></table>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
