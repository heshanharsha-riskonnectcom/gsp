1. # Azure DevOps Agent Upgrade

## Copy vsts-agent-win-x64-2.220.0.zip to the server as C:\temp\vsts-agent-win-x64-2.220.0.zip.

## 1. Find the agents on the server

```powershell
Get-CimInstance Win32_Service -Filter "Name like 'vstsagent%'" | Select Name, State, StartName, PathName | Format-List
```

## 2. Upgrade the agent

```powershell
$svc = "<service name from step 1> ex: vstsagent.cma-devops.Camms.Risk-CMA-Slsvr20.CMA-SLSVR20"
$dir = '<agent directory> ex: C:\azagent\A1'

Stop-Service $svc
robocopy $dir "$dir.bak-20261123" /E /XD "$dir\_work" /NFL /NDL /NP
Expand-Archive C:\temp\vsts-agent-win-x64-2.220.0.zip -DestinationPath $dir -Force
Start-Service $svc
& "$dir\bin\Agent.Listener.exe" --version
```
