
# Instructions to Download, Unblock, and Run a PowerShell Script

## Step 1: Download the `.ps1` File and `.crt` File
1. Download the PowerShell script (`.ps1` file) and the certificate file (`.crt`) from the source.
   - Save these files to a directory on your machine, such as `C:\Tmp`.

   Example:
   - Save the PowerShell script as `Install-Certificate.ps1`.
   - Save the certificate file as `stonecodelabs-prod.crt`.

## Step 2: Unblock the `.ps1` File
When you download files from external sources (like the internet or email), Windows may block them. To unblock the `.ps1` file:

1. Open **PowerShell** as Administrator.
   - To do this, press `Start`, type **PowerShell**, right-click **Windows PowerShell**, and select **Run as Administrator**.
2. Run the following command to unblock the PowerShell script:

   ```powershell
   Unblock-File -Path "C:\Tmp\Install-Certificate.ps1"
   ```

   This will remove the "blocked" status from the file.

## Step 3: Check the Current PowerShell Execution Policy
Before running the PowerShell script, you need to ensure that your execution policy allows for script execution.

1. Run the following command in PowerShell to check your current execution policy:

   ```powershell
   Get-ExecutionPolicy -List
   ```

   This will show you the execution policies for different scopes (Machine, User, Process, etc.).

## Step 4: Set Execution Policy to `RemoteSigned`
If the execution policy is too restrictive (e.g., `Restricted`), you can change it to `RemoteSigned`. This allows locally created scripts to run, while scripts from the internet must be signed.

1. To set the execution policy to `RemoteSigned`, run the following command in PowerShell (as Administrator):

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

2. If you want to set it system-wide, you can set it for the local machine:

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope LocalMachine
   ```

3. Confirm the change by typing `Y` (Yes) if prompted.

## Step 5: Run the PowerShell Script
Now that the execution policy is set and the script is unblocked, you can run the script.

1. Navigate to the directory where your `.ps1` file is located, for example:

   ```powershell
   cd C:\Tmp
   ```

2. Run the PowerShell script:

   ```powershell
   ./Install-Certificate.ps1
   ```

   If your script requires the `.crt` file, make sure it is referenced correctly within the script or in the command execution.

## Optional: Reset the Execution Policy (If Needed)
If you want to restore your execution policy to a more restrictive setting after running the script, you can reset it using:

```powershell
Set-ExecutionPolicy Restricted -Scope CurrentUser
```

Or for system-wide:

```powershell
Set-ExecutionPolicy Restricted -Scope LocalMachine
```

## Summary of Commands

```powershell
# Unblock the .ps1 file
Unblock-File -Path "C:\Tmp\Install-Certificate.ps1"

# Check current execution policy
Get-ExecutionPolicy -List

# Set execution policy to RemoteSigned
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Navigate to the script directory
cd C:\Tmp

# Run the .ps1 script
./Install-Certificate.ps1

# (Optional) Reset execution policy to Restricted
Set-ExecutionPolicy Restricted -Scope CurrentUser
```
