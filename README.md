# CloudPC GitHub Workflows Documentation

The `main.yml` file within the `.github/workflows` directory defines a GitHub Actions workflow for the CloudPC repository. Here is an extensive documentation of what the `main.yml` file does and its place within the CI/CD pipeline.

## Overview of the `main.yml` File

The `main.yml` file orchestrates a Continuous Integration (CI) pipeline that automatically executes a series of steps every time a push event occurs or the workflow is manually triggered (through `workflow_dispatch`). The current configuration is designed to run on a Windows virtual machine provided by GitHub Actions (`windows-latest`).

## Workflow Structure

The workflow is named "CI", and it comprises a single job identified as `build`. The job contains several steps that execute sequentially whenever the workflow is triggered.

### Triggers

*   `push`: This initiates the workflow whenever code is pushed to the repository.
    
*   `workflow_dispatch`: This allows the workflow to be manually triggered from GitHub's UI.
    

### Job: `build`

*   **Environment**: The job runs on the latest Windows runner provided by GitHub (`windows-latest`).
    

### Steps in the `build` Job

1.  **Download**: This step uses PowerShell's `Invoke-WebRequest` cmdlet to download `ngrok` - a tool that exposes local servers to the public internet by creating a secure tunnel.
    
    ```yaml
    - name: Download
      run: Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
    
    ```
    
2.  **Extract**: Extracts the downloaded ngrok.zip file using PowerShell's `Expand-Archive` cmdlet.
    
    ```yaml
    - name: Extract
      run: Expand-Archive ngrok.zip
    
    ```
    
3.  **Auth**: Authenticates the ngrok client using a secret token (`NGROK_AUTH_TOKEN`) stored in GitHub Secrets.
    
    ```yaml
    - name: Auth
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    
    ```
    
4.  **Enable TS**: Enables the Terminal Services (Remote Desktop) on the Windows runner by modifying registry values.
    
    ```yaml
    - name: Enable TS
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
    
    ```
    
5.  Enables the necessary firewall rule to allow Remote Desktop connections.
    
    ```yaml
    - run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    
    ```
    
6.  Sets the policy to require network-level authentication for Remote Desktop connections.
    
    ```yaml
    - run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    
    ```
    
7.  Sets a password for the `runneradmin` user, which is required for Remote Desktop connections.
    
    ```yaml
    - run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)
    
    ```
    
8.  **Create Tunnel**: The final step in the job is to create an ngrok tunnel to the RDP port (port 3389) so that the Windows environment is accessible over the network.
    
    ```yaml
    - name: Create Tunnel
      run: .\ngrok\ngrok.exe tcp 3389
    
    ```
    

## Security Considerations

This workflow requires the storage of sensitive data in GitHub Secrets, specifically `NGROK_AUTH_TOKEN`. It's critical that this secret token is kept confidential to prevent unauthorized access.

The use of an RDP password (`P@ssw0rd!`) is demonstrated, which should be changed and stored securely if used in production.

## Usage and Adaptation

This workflow may be used or adapted by developers who need to enable RDP access to their GitHub Actions Windows runner for debugging or direct interaction with the CI environment. It automates the process of setting up an RDP server with `ngrok` enabling secure access from any location.
