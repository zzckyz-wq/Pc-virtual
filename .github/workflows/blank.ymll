name: PC criado pelo @alves7.pcm

on:
  workflow_dispatch:
    inputs:
      instance_number:
        description: 'Instance Number (1-10)'
        required: true
        default: '1'
        type: string

jobs:
  deploy-rdp:
    runs-on: windows-latest
    timeout-minutes: 360
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
      
    - name: System Information
      run: |
        Write-Host "=== ENIGMANO RDP DEPLOYMENT ===" -ForegroundColor Green
        Write-Host "Instance Number: ${{ github.event.inputs.instance_number }}" -ForegroundColor Yellow
        Write-Host "Runner: $env:RUNNER_NAME" -ForegroundColor Yellow
        Write-Host "Workflow: $env:GITHUB_WORKFLOW" -ForegroundColor Yellow
        
        Write-Host "`n=== SYSTEM SPECIFICATIONS ===" -ForegroundColor Cyan
        Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, TotalPhysicalMemory
        Get-WmiObject -Class Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors
        
    - name: Enable RDP
      run: |
        Write-Host "Enabling Remote Desktop..." -ForegroundColor Green
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        
    - name: Create User Account
      run: |
        Write-Host "Setting up user account..." -ForegroundColor Green
        $Password = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
        Set-LocalUser -Name "runneradmin" -Password $Password
        Add-LocalGroupMember -Group "Remote Desktop Users" -Member "runneradmin"
        Write-Host "Username: runneradmin" -ForegroundColor Yellow
        Write-Host "Password: P@ssw0rd!" -ForegroundColor Yellow
        
    - name: Install Tailscale (Silent)
      run: |
        Write-Host "Downloading Tailscale..." -ForegroundColor Green
        $tsUrl = "https://pkgs.tailscale.com/stable/tailscale-setup-1.82.0-amd64.msi"
        $installerPath = "$env:TEMP\tailscale.msi"
        Invoke-WebRequest -Uri $tsUrl -OutFile $installerPath
        Write-Host "Installing Tailscale silently..." -ForegroundColor Green
        Start-Process msiexec.exe -ArgumentList "/i `"$installerPath`" /quiet /norestart" -Wait
        Remove-Item $installerPath -Force
        Write-Host "Tailscale installed successfully!" -ForegroundColor Green
        
    - name: Start Tailscale
      run: |
        Write-Host "Starting Tailscale..." -ForegroundColor Green
        & "C:\Program Files\Tailscale\tailscale.exe" up --authkey ${{ secrets.TAILSCALE_AUTH_KEY }} --hostname "enigmano-${{ github.run_id }}"
        Start-Sleep -Seconds 15
        
    - name: Get Tailscale IP
      run: |
        $tsIP = & "C:\Program Files\Tailscale\tailscale.exe" ip -4
        Write-Host "Tailscale IP: $tsIP" -ForegroundColor Green
        echo "TAILSCALE_IP=$tsIP" >> $env:GITHUB_ENV
        
    - name: Install Google Chrome (Direct)
      run: |
        Write-Host "Downloading Google Chrome..." -ForegroundColor Green
        $chromeUrl = "https://dl.google.com/chrome/install/latest/chrome_installer.exe"
        $installerPath = "$env:TEMP\chrome_installer.exe"
        Invoke-WebRequest -Uri $chromeUrl -OutFile $installerPath
        Write-Host "Installing Google Chrome silently..." -ForegroundColor Green
        Start-Process -FilePath $installerPath -ArgumentList "/silent /install" -Wait
        Remove-Item $installerPath -Force
        Write-Host "Google Chrome installed!" -ForegroundColor Green
        
    - name: Install 7-Zip (Direct)
      run: |
        Write-Host "Downloading 7-Zip..." -ForegroundColor Green
        $zipUrl = "https://www.7-zip.org/a/7z2409-x64.exe"
        $installerPath = "$env:TEMP\7z-installer.exe"
        Invoke-WebRequest -Uri $zipUrl -OutFile $installerPath
        Write-Host "Installing 7-Zip silently..." -ForegroundColor Green
        Start-Process -FilePath $installerPath -ArgumentList "/S" -Wait
        Remove-Item $installerPath -Force
        Write-Host "7-Zip installed!" -ForegroundColor Green
        
    - name: Connection Details
      run: |
        Write-Host "`n=== CONNECTION DETAILS (TAILSCALE) ===" -ForegroundColor Green
        Write-Host "╔══════════════════════════════════════╗" -ForegroundColor Cyan
        Write-Host "║         ENIGMANO RDP ACCESS          ║" -ForegroundColor Cyan  
        Write-Host "╠══════════════════════════════════════╣" -ForegroundColor Cyan
        Write-Host "║ Host: $env:TAILSCALE_IP              ║" -ForegroundColor White
        Write-Host "║ Username: runneradmin                ║" -ForegroundColor White
        Write-Host "║ Password: P@ssw0rd!                  ║" -ForegroundColor White
        Write-Host "╚══════════════════════════════════════╝" -ForegroundColor Cyan
        
        Write-Host "`nConnection Instructions:" -ForegroundColor Yellow
        Write-Host "1. Install Tailscale on your device (phone, tablet, Mac, Linux)" -ForegroundColor White
        Write-Host "2. Log in with the same Tailscale account" -ForegroundColor White
        Write-Host "3. Open Remote Desktop app and enter the IP above" -ForegroundColor White
        Write-Host "4. Use the provided credentials" -ForegroundColor White
        
    - name: Keep Session Alive
      run: |
        Write-Host "`n=== SESSION ACTIVE ===" -ForegroundColor Green
        Write-Host "EnigMano RDP instance is now running!" -ForegroundColor Cyan
        Write-Host "Instance will remain active for up to 6 hours." -ForegroundColor Yellow
        Write-Host "Do not close this window to maintain the connection." -ForegroundColor Red
        
        $counter = 0
        while ($true) {
          $counter++
          Write-Host "Session active - Heartbeat #$counter ($(Get-Date))" -ForegroundColor Green
          
          $tsStatus = & "C:\Program Files\Tailscale\tailscale.exe" status
          if ($tsStatus -match "active") {
            Write-Host "Tailscale status: ACTIVE" -ForegroundColor Green
          } else {
            Write-Host "Tailscale status: ERROR - Restarting..." -ForegroundColor Red
            & "C:\Program Files\Tailscale\tailscale.exe" up --authkey ${{ secrets.TAILSCALE_AUTH_KEY }} --hostname "enigmano-${{ github.run_id }}"
          }
          
          Start-Sleep -Seconds 60
        }
