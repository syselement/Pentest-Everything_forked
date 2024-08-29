# Nmap Commands for port discovery



## Description

Nice and easy Nmap port scans for identifying open ports and outputting into a list of IP addresses.

{% code overflow="wrap" %}
```bash
# One liner for scanning addresses from file and displaying URL addresses in output
nmap -p 80,443,8080,8443,8000 --open -oG - -iL CIDR.txt | grep "/open" | awk '/80\/open/ {print "http://" $2 ":80"} /443\/open/ {print "https://" $2 ":443"} /8080\/open/ {print "http://" $2 ":8080"} /8443\/open/ {print "https://" $2 ":8443"} /8000\/open/ {print "http://" $2 ":8000"}'

# Windows
$results = .\nmap.exe -p 80,443,8080,8443,8000 --open -oG - -iL targets.txt; $results | ForEach-Object { if ($_ -match "Host: (\d+\.\d+\.\d+\.\d+)") { $ip = $matches[1] }; if ($ip) { if ($_ -match "80/open") { "http://{0}:80" -f $ip }; if ($_ -match "443/open") { "https://{0}:443" -f $ip }; if ($_ -match "8080/open") { "http://{0}:8080" -f $ip }; if ($_ -match "8443/open") { "https://{0}:8443" -f $ip }; if ($_ -match "8000/open") { "http://{0}:8000" -f $ip } } }
```
{% endcode %}

{% code overflow="wrap" %}
```bash
# Scan for SSH (Port 22)
nmap <CIDR> -p 22 --open -oN SSH-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' SSH-ports.log | sort | uniq > SSH-Ports.txt && rm SSH-ports.log

# Scan for Telnet (Port 23)
nmap <CIDR> -p 23 --open -oN Telnet-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' Telnet-ports.log | sort | uniq > Telnet-Ports.txt && rm Telnet-ports.log

# Scan for FTP (Port 21)
nmap <CIDR> -p 21 --open -oN FTP-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' FTP-ports.log | sort | uniq > FTP-Ports.txt && rm FTP-ports.log

# Scan for SNMP (UDP Port 161)
sudo nmap <CIDR> -sU -p 161 --open -oN SNMP-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' SNMP-ports.log | sort | uniq > SNMP-Ports.txt && rm SNMP-ports.log

# Scan for IPMI (UDP Port 623)
sudo nmap <CIDR> -sU -p 623 --open -oN IPMI-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' IPMI-ports.log | sort | uniq > IPMI-Ports.txt && rm IPMI-ports.log

# Scan for HTTP (Port 80)
nmap <CIDR> -p 80 --open -oN HTTP-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' HTTP-ports.log | sort | uniq > HTTP-Ports.txt && rm HTTP-ports.log

# Scan for HTTPS (Port 443)
nmap <CIDR> -p 443 --open -oN HTTPS-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' HTTPS-ports.log | sort | uniq > HTTPS-Ports.txt && rm HTTPS-ports.log

# Scan for SMB (Port 445)
nmap <CIDR> -p 445 --open -oN SMB-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' SMB-ports.log | sort | uniq > SMB-Ports.txt && rm SMB-ports.log

# Scan for VNC (Port 5900)
nmap <CIDR> -p 5900 --open -oN VNC-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' VNC-ports.log | sort | uniq > VNC-Ports.txt

# Scan for NFS (Port 2049)
nmap <CIDR> -p 2049 --open -oN NFS-ports.log && awk '/Nmap scan report for/ {gsub(/[()]/, "", $NF); print $NF}' NFS-ports.log | sort | uniq > NFS-Ports.txt
```
{% endcode %}



Similar as above but for Windows, using PowerShell and reading from Targets.txt and outputting to $HOME.

{% code overflow="wrap" %}
```powershell
# Scan for SSH (Port 22)
.\nmap.exe -iL targets.txt -p 22 --open -oN "$HOME\SSH-ports.log"; Get-Content "$HOME\SSH-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\SSH-Ports.txt"; Remove-Item "$HOME\SSH-ports.log"

# Scan for Telnet (Port 23)
.\nmap.exe -iL targets.txt -p 23 --open -oN "$HOME\Telnet-ports.log"; Get-Content "$HOME\Telnet-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\Telnet-Ports.txt"; Remove-Item "$HOME\Telnet-ports.log"

# Scan for FTP (Port 21)
.\nmap.exe -iL targets.txt -p 21 --open -oN "$HOME\FTP-ports.log"; Get-Content "$HOME\FTP-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\FTP-Ports.txt"; Remove-Item "$HOME\FTP-ports.log"

# Scan for SNMP (UDP Port 161)
.\nmap.exe -iL targets.txt -sU -p 161 --open -oN "$HOME\SNMP-ports.log"; Get-Content "$HOME\SNMP-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\SNMP-Ports.txt"; Remove-Item "$HOME\SNMP-ports.log"

# Scan for IPMI (UDP Port 623)
.\nmap.exe -iL targets.txt -sU -p 623 --open -oN "$HOME\IPMI-ports.log"; Get-Content "$HOME\IPMI-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\IPMI-Ports.txt"; Remove-Item "$HOME\IPMI-ports.log"

# Scan for HTTP (Port 80)
.\nmap.exe -iL targets.txt -p 80 --open -oN "$HOME\HTTP-ports.log"; Get-Content "$HOME\HTTP-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\HTTP-Ports.txt"; Remove-Item "$HOME\HTTP-ports.log"

# Scan for HTTPS (Port 443)
.\nmap.exe -iL targets.txt -p 443 --open -oN "$HOME\HTTPS-ports.log"; Get-Content "$HOME\HTTPS-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\HTTPS-Ports.txt"; Remove-Item "$HOME\HTTPS-ports.log"

# Scan for SMB (Port 445)
.\nmap.exe -iL targets.txt -p 445 --open -oN "$HOME\SMB-ports.log"; Get-Content "$HOME\SMB-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\SMB-Ports.txt"; Remove-Item "$HOME\SMB-ports.log"

# Scan for VNC (Port 5900)
.\nmap.exe -iL targets.txt -p 5900 --open -oN "$HOME\VNC-ports.log"; Get-Content "$HOME\VNC-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\VNC-Ports.txt"; Remove-Item "$HOME\VNC-ports.log"

# Scan for NFS (Port 2049)
.\nmap.exe -iL targets.txt -p 2049 --open -oN "$HOME\NFS-ports.log"; Get-Content "$HOME\NFS-ports.log" | Select-String "Nmap scan report for" | ForEach-Object { $_ -replace ".*for ", "" -replace "[()]", "" } | Sort-Object -Unique | Set-Content "$HOME\NFS-Ports.txt"; Remove-Item "$HOME\NFS-ports.log"
```
{% endcode %}
