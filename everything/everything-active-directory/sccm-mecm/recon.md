# Recon

**Common SCCM Ports**

{% embed url="https://learn.microsoft.com/en-us/mem/configmgr/core/plan-design/hierarchy/ports" %}

<table><thead><tr><th width="118">Protocol</th><th width="144">Port(s)</th><th>Service</th></tr></thead><tbody><tr><td>TCP</td><td>8530, 8531,</td><td>Site Server, Management Point</td></tr><tr><td></td><td>10123</td><td></td></tr><tr><td>TCP</td><td>49152-49159</td><td>Distribution Point</td></tr><tr><td>UDP</td><td>4011</td><td>Operating System Deployment (OSD)</td></tr></tbody></table>

Nmap

```
# SCCM search
nmap -p 80,443,445,1433,10123,8530,8531 -sV [IP]

# search PXE
nmap -p 67,68,69,4011,547 -sV -sU [IP]
```



### PowerShell

```
([ADSISearcher]("objectClass=mSSMSManagementPoint")).FindAll() | % {$_.Properties}
```

## Configuration Manager

<div align="center" data-full-width="true">

<figure><img src="../../../.gitbook/assets/image (2141).png" alt=""><figcaption></figcaption></figure>

</div>

## Linux Recon

* sccmhunter: [https://github.com/garrettfoster13/sccmhunter](https://github.com/garrettfoster13/sccmhunter)

```python
python3 sccmhunter.py find -u <user> -p <password> -d <domain> -dc-ip <ip> -debug

# Review collected information
python3 sccmhunter.py show -all 
```

Discovery through SMB

```python
smbmap -u <user> -p <password> -d <domain> -H <ip> 
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
