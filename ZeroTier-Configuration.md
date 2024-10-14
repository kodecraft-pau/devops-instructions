
# ZeroTier Configuration Script

This PowerShell script allows you to easily join a ZeroTier network and enable DNS for that network. The script automates the process of configuring ZeroTier without requiring you to enter the DNS server IP.

## Prerequisites

- **ZeroTier must be installed** on your system. You can download it from [ZeroTier's official site](https://www.zerotier.com/download/).
- **Windows PowerShell** (preferably run as Administrator).

## Setup Instructions

### 1. Unblock the PowerShell Script

If you downloaded the `ZeroTier-Configuration.ps1` file from the internet, it might be blocked by Windows. To unblock it, follow these steps:

1. Open PowerShell as Administrator.
2. Run the following command to unblock the script:
   ```powershell
   Unblock-File -Path C:\Tmp\ZeroTier-Configuration.ps1
   ```

### 2. Set Execution Policy to RemoteSigned

If your system’s execution policy is too restrictive (e.g., **Restricted**), you may need to change it to allow running locally created scripts. The recommended policy is **RemoteSigned**, which allows running locally created scripts but requires signed scripts from remote sources.

To temporarily change the execution policy for the current session:

1. Open PowerShell as Administrator.
2. Run the following command:
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope Process -Force
   ```

This will allow PowerShell scripts to run only during this session. If you want to change the policy permanently for your user, you can omit the `-Scope Process`.

### 3. Running the Script

Once the script is unblocked and the execution policy is set:

1. Navigate to the directory where the `ZeroTier-Configuration.ps1` script is located:
   ```powershell
   cd C:\Tmp
   ```

2. Run the script:
   ```powershell
   ./ZeroTier-Configuration.ps1
   ```

### 4. Script Workflow

- The script will prompt you to enter your **ZeroTier Network ID**.
- It will check if the system is already connected to the network. If not, it will attempt to join the network.
- After joining, it will automatically enable DNS for that network.

## Troubleshooting

- If you receive an error about running scripts being disabled, make sure you have set the execution policy to `RemoteSigned`.
- If ZeroTier is not installed, download and install it from [ZeroTier's official site](https://www.zerotier.com/download/).

## License

This script is provided "as is" without any warranties. Feel free to modify and use it as needed.
