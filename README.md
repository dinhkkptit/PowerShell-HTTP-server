# PowerShell HTTP Servers

This repository contains two standalone PowerShell implementations of lightweight HTTP servers based on the .NET HttpListener class.

## Contents
- `SimplePowerShellHTTPServer/` — single-script module exposing `Start-HttpServer` with static file serving, uploads/downloads, and API-key-protected remote command execution.
- `WebServer/` — module exposing `Start-Webserver` with static content, script execution, PSP pages, and upload/download utilities.

## Prerequisites
- Windows PowerShell 4.0+ (PowerShell 5.x or later recommended) on Windows.
- Run the console as Administrator when binding to ports below 1024 or to `http://+:port/`.
- If scripts are blocked, enable them for the session: `Set-ExecutionPolicy -Scope Process RemoteSigned`.
- Open your firewall for the chosen port when exposing the server to other machines (for example: `netsh advfirewall firewall add rule name="PowerShellHttp" dir=in action=allow protocol=TCP localport=8080`).

## SimplePowerShellHTTPServer quick start
1. Import the module:
```powershell
Import-Module .\SimplePowerShellHTTPServer\SimplePowerShellHttpServer.psm1
```
2. Start the server (8080 to avoid privileged ports):
```powershell
Start-HttpServer -URL 'localhost' -Port 8080 -Path (Join-Path $PWD 'SimplePowerShellHTTPServer')
```
   - A random API key prints at startup; use it for remote commands. Logs go to `SPHS.log` when `-LogLevel` is set. The default index page in the module folder is served.
3. Serve your own content by pointing `-Path` at another directory or load settings from JSON with `Start-HttpServer -Load .\server.json`.

Common actions:
```powershell
# Download a file
curl http://localhost:8080/foo.txt --output foo.txt

# Upload via CLI (Name header required)
Invoke-WebRequest -Method PUT -Uri http://localhost:8080 -InFile .\bar.txt -Headers @{Name='bar.txt'}

# Remote command execution (requires API key from startup)
curl http://localhost:8080 -H "Action: CommandExecute" -H "Command: Get-Process" -H "x-api-key: <key>"
```
For all parameters, run `Get-Help Start-HttpServer -Full` or see `SimplePowerShellHTTPServer/README.md`.

## WebServer module quick start
1. Import the module:
```powershell
Import-Module .\WebServer\Module\WebServer.psd1
```
2. Start locally on port 8080:
```powershell
Start-Webserver        # binds http://localhost:8080/
```
3. Expose a folder to all interfaces (admin required for `+` binding):
```powershell
Start-Webserver "http://+:8080/" "C:\Data"
```
Features:
- Static file serving from the script/base directory.
- PSP pages (`.psp`) with embedded PowerShell in HTML (see `WebServer/helloworld.psp`).
- Command/script execution plus upload/download endpoints and basic status/log views.

More details and screenshots: `WebServer/README.md`. HTTPS setup instructions: `WebServer/https.md`.

## Notes
- Both servers are for local testing or demos only; there is no built-in security or authentication.
- Stop the process (Ctrl+C) when done and remove any temporary firewall rules you added.
