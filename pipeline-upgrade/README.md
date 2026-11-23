1. # Azure DevOps Agent Upgrade

## 1. Find the agents on the server

```powershell
Get-CimInstance Win32_Service -Filter "Name like 'vstsagent%'" | Select Name, State, StartName, PathName | Format-List
```

The server is expected to have two agents:

- One for CMA-Slsvr20 (A1).
- One for the deployment group CMA-SLSVR20-agent-2 (A2).

Upgrade them individually, starting with A1.

## 2. Verify and unblock the agent package

```powershell
(Get-FileHash C:\temp\vsts-agent-win-x64-2.220.0.zip -Algorithm SHA256).Hash
```

The result must be 56680A007E970279DC97C65E0EC781838F88CBFBCE334BCF307706A6A9575210.

```powershell
Unblock-File C:\temp\vsts-agent-win-x64-2.220.0.zip
```

## 3. Upgrade the agent

Set the variables below to the A1 service name and agent directory:

```powershell
$svc = "<service name from step 1, for A1>"
$dir = 'C:\azagent\A1'

Stop-Service $svc
robocopy $dir "$dir.bak-20261123" /E /XD "$dir\_work" /NFL /NDL /NP
Expand-Archive C:\temp\vsts-agent-win-x64-2.220.0.zip -DestinationPath $dir -Force
Start-Service $svc
& "$dir\bin\Agent.Listener.exe" --version
```

- `robocopy` creates a backup of the agent folder, excluding the large `_work` folder.
- `Expand-Archive` replaces only the program files.
- Agent settings and registration are not included in the archive, so they remain unchanged.
- The final command should print `2.220.0`.

## 4. Verify the upgrade

1. Wait approximately one minute, then reload the Azure DevOps agent page.
2. Confirm that the agent version is `2.220.0` and that the agent is online.
3. Confirm that the **Capabilities** tab shows `Agent.Version` as `2.220.0`.
4. If the agent remains offline, inspect the newest log:

   ```powershell
   C:\azagent\A1\_diag\Agent_*.log
   ```

5. Redeploy the last known-good release to this server, such as **Release-407 / Dev WinAuth** in **Camms.Risk-Dev-CD**, and confirm that it succeeds.

Repeat step 3 for the second agent using the A2 service name and:

```powershell
$dir = 'C:\azagent\A2'
```

## Rollback

```powershell
Stop-Service $svc
robocopy "$dir.bak-20261123" $dir /E
Start-Service $svc
```

This restores the backed-up `2.181.2` files and starts the agent with the previous installation.