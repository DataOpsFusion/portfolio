# Windows Administration

Master Windows system administration for modern environments. Learn PowerShell, Active Directory, and Windows Server management.

## Why Windows Administration Matters

Even in a Linux-heavy world, Windows systems are everywhere:
- **Desktop environments**: Most developers use Windows workstations
- **Enterprise applications**: Many business apps run on Windows
- **Hybrid environments**: Mixed Windows/Linux infrastructures
- **Legacy systems**: Existing Windows infrastructure needs management

## PowerShell Fundamentals

### Getting Started with PowerShell
```powershell
# Check PowerShell version
$PSVersionTable

# Get help for any command
Get-Help Get-Process -Examples

# Find commands
Get-Command *Service*

# Get object properties
Get-Service | Get-Member
```

### Object-Oriented Approach
Unlike traditional shells, PowerShell works with objects:
```powershell
# Get processes and work with objects
$processes = Get-Process
$processes | Where-Object { $_.CPU -gt 100 }
$processes | Sort-Object CPU -Descending | Select-Object -First 5

# Work with services
Get-Service | Where-Object { $_.Status -eq 'Running' } | 
    Select-Object Name, Status, StartType
```

### Variables and Data Types
```powershell
# Variables
$name = "Server01"
$port = 443
$isOnline = $true

# Arrays
$servers = @("Server01", "Server02", "Server03")
$numbers = 1..10

# Hash tables
$serverInfo = @{
    Name = "Server01"
    IP = "192.168.1.100"
    OS = "Windows Server 2022"
}
```

### Functions and Scripts
```powershell
# Simple function
function Get-DiskSpace {
    param(
        [string]$ComputerName = $env:COMPUTERNAME
    )
    
    Get-WmiObject -Class Win32_LogicalDisk -ComputerName $ComputerName |
        Select-Object DeviceID, 
                     @{Name="Size(GB)";Expression={[math]::Round($_.Size/1GB,2)}},
                     @{Name="FreeSpace(GB)";Expression={[math]::Round($_.FreeSpace/1GB,2)}},
                     @{Name="PercentFree";Expression={[math]::Round(($_.FreeSpace/$_.Size)*100,2)}}
}

# Advanced function with parameters
function Install-Software {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$SoftwareName,
        
        [Parameter(Mandatory=$true)]
        [string]$InstallerPath,
        
        [string[]]$ComputerName = $env:COMPUTERNAME,
        
        [switch]$Silent
    )
    
    foreach ($Computer in $ComputerName) {
        Write-Verbose "Installing $SoftwareName on $Computer"
        
        if ($Silent) {
            $arguments = "/S"
        } else {
            $arguments = ""
        }
        
        Invoke-Command -ComputerName $Computer -ScriptBlock {
            Start-Process -FilePath $using:InstallerPath -ArgumentList $using:arguments -Wait
        }
    }
}
```

## Active Directory Management

### User Management
```powershell
# Import Active Directory module
Import-Module ActiveDirectory

# Create new user
New-ADUser -Name "John Doe" -GivenName "John" -Surname "Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@company.com" -Path "OU=Users,DC=company,DC=com" -AccountPassword (ConvertTo-SecureString "TempPassword123!" -AsPlainText -Force) -Enabled $true

# Bulk user creation from CSV
$users = Import-Csv -Path "C:\Users\new_users.csv"
foreach ($user in $users) {
    New-ADUser -Name "$($user.FirstName) $($user.LastName)" -GivenName $user.FirstName -Surname $user.LastName -SamAccountName $user.Username -UserPrincipalName "$($user.Username)@company.com" -Path $user.OU -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) -Enabled $true
}

# Modify user properties
Set-ADUser -Identity "jdoe" -Department "IT" -Title "System Administrator"

# Disable inactive users
$inactiveUsers = Search-ADAccount -AccountInactive -TimeSpan (New-TimeSpan -Days 90) -UsersOnly
$inactiveUsers | Disable-ADAccount
```

### Group Management
```powershell
# Create security group
New-ADGroup -Name "IT Support" -GroupScope Global -GroupCategory Security -Path "OU=Groups,DC=company,DC=com"

# Add users to group
Add-ADGroupMember -Identity "IT Support" -Members "jdoe", "asmith"

# Get group membership
Get-ADGroupMember -Identity "IT Support" | Select-Object Name, SamAccountName

# Find groups a user belongs to
Get-ADUser -Identity "jdoe" -Properties MemberOf | Select-Object -ExpandProperty MemberOf
```

### Organizational Units (OUs)
```powershell
# Create OU structure
New-ADOrganizationalUnit -Name "Company" -Path "DC=company,DC=com"
New-ADOrganizationalUnit -Name "Departments" -Path "OU=Company,DC=company,DC=com"
New-ADOrganizationalUnit -Name "IT" -Path "OU=Departments,OU=Company,DC=company,DC=com"
New-ADOrganizationalUnit -Name "Sales" -Path "OU=Departments,OU=Company,DC=company,DC=com"

# Move users to appropriate OUs
Get-ADUser -Filter "Department -eq 'IT'" | Move-ADObject -TargetPath "OU=IT,OU=Departments,OU=Company,DC=company,DC=com"
```

## Group Policy Management

### Creating and Linking GPOs
```powershell
# Import Group Policy module
Import-Module GroupPolicy

# Create new GPO
New-GPO -Name "Desktop Security Policy" -Comment "Security settings for desktop computers"

# Link GPO to OU
New-GPLink -Name "Desktop Security Policy" -Target "OU=Desktops,OU=Company,DC=company,DC=com"

# Set registry value via GPO
Set-GPRegistryValue -Name "Desktop Security Policy" -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" -ValueName "EnableLUA" -Type DWord -Value 1

# Import security template
Import-GPO -BackupGpoName "Security Template" -Path "C:\GPOBackups" -TargetName "Desktop Security Policy"
```

### GPO Reporting
```powershell
# Generate GPO report
Get-GPOReport -Name "Desktop Security Policy" -ReportType Html -Path "C:\Reports\GPOReport.html"

# Get all GPOs linked to an OU
Get-GPInheritance -Target "OU=IT,OU=Departments,OU=Company,DC=company,DC=com"

# Find unlinked GPOs
Get-GPO -All | Where-Object { $_.GpoStatus -eq "AllSettingsDisabled" }
```

## Windows Server Management

### Server Roles and Features
```powershell
# List available features
Get-WindowsFeature

# Install IIS
Install-WindowsFeature -Name IIS-WebServerRole -IncludeManagementTools

# Install Active Directory Domain Services
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote server to domain controller
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "company.com" -InstallDns -SafeModeAdministratorPassword (ConvertTo-SecureString "SafeModePassword123!" -AsPlainText -Force)
```

### Windows Updates
```powershell
# Install PSWindowsUpdate module
Install-Module PSWindowsUpdate

# Check for updates
Get-WUList

# Install all updates
Get-WUInstall -AcceptAll -AutoReboot

# Install specific updates
Get-WUInstall -KBArticleID KB5000802 -AcceptAll

# Schedule update installation
Get-WUInstall -AcceptAll -ScheduleJob (Get-Date).AddHours(2)
```

### Event Log Management
```powershell
# Get recent errors from System log
Get-EventLog -LogName System -EntryType Error -Newest 10

# Get specific events
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 50

# Clear event logs
Clear-EventLog -LogName Application

# Create custom event log
New-EventLog -LogName "CustomApp" -Source "MyApplication"
Write-EventLog -LogName "CustomApp" -Source "MyApplication" -EventId 1001 -EntryType Information -Message "Application started successfully"
```

## Registry Management

### Reading and Writing Registry
```powershell
# Read registry value
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" -Name "ProductName"

# Create registry key
New-Item -Path "HKLM:\SOFTWARE\MyApp" -Force

# Set registry value
Set-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" -Name "Version" -Value "1.0.0"

# Remove registry key
Remove-Item -Path "HKLM:\SOFTWARE\MyApp" -Recurse

# Export registry key
reg export "HKLM\SOFTWARE\MyApp" "C:\Backup\MyApp.reg"

# Import registry file
reg import "C:\Backup\MyApp.reg"
```

## File System Management

### NTFS Permissions
```powershell
# Get ACL for a folder
Get-Acl -Path "C:\SharedFolder"

# Set permissions
$acl = Get-Acl -Path "C:\SharedFolder"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("DOMAIN\User", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($accessRule)
Set-Acl -Path "C:\SharedFolder" -AclObject $acl

# Remove permissions
$acl.RemoveAccessRule($accessRule)
Set-Acl -Path "C:\SharedFolder" -AclObject $acl
```

### Disk Management
```powershell
# Get disk information
Get-Disk

# Initialize new disk
Initialize-Disk -Number 1 -PartitionStyle GPT

# Create partition
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter D

# Format volume
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "Data"

# Resize partition
Resize-Partition -DriveLetter D -Size 500GB
```

## Performance Monitoring

### Performance Counters
```powershell
# Get CPU usage
Get-Counter "\Processor(_Total)\% Processor Time"

# Monitor multiple counters
$counters = @(
    "\Processor(_Total)\% Processor Time",
    "\Memory\Available MBytes",
    "\PhysicalDisk(_Total)\% Disk Time"
)
Get-Counter -Counter $counters -SampleInterval 5 -MaxSamples 10

# Create custom performance counter set
$counterSetName = "MyApp Performance"
$counters = @("Requests per Second", "Response Time")
```

### System Information
```powershell
# Get system information
Get-ComputerInfo

# Get installed software
Get-WmiObject -Class Win32_Product | Select-Object Name, Version, Vendor

# Get running processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Get network configuration
Get-NetIPConfiguration
Get-NetAdapter | Where-Object Status -eq "Up"
```

## Automation and Scripting

### Scheduled Tasks
```powershell
# Create scheduled task
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\Backup.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At "2:00AM"
$settings = New-ScheduledTaskSettingsSet -ExecutionTimeLimit (New-TimeSpan -Hours 2)
Register-ScheduledTask -TaskName "Daily Backup" -Action $action -Trigger $trigger -Settings $settings -User "SYSTEM"

# Modify scheduled task
Set-ScheduledTask -TaskName "Daily Backup" -Trigger (New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday,Wednesday,Friday -At "3:00AM")

# Run scheduled task
Start-ScheduledTask -TaskName "Daily Backup"
```

### Remote Management
```powershell
# Enable PowerShell remoting
Enable-PSRemoting -Force

# Execute commands on remote computers
Invoke-Command -ComputerName "Server01", "Server02" -ScriptBlock { Get-Service | Where-Object Status -eq "Stopped" }

# Create persistent session
$session = New-PSSession -ComputerName "Server01"
Invoke-Command -Session $session -ScriptBlock { Import-Module ActiveDirectory }
Invoke-Command -Session $session -ScriptBlock { Get-ADUser -Filter * }
```

## Security Best Practices

### User Account Control (UAC)
```powershell
# Check UAC status
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA"

# Configure UAC levels
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 2
```

### Windows Defender
```powershell
# Get Windows Defender status
Get-MpComputerStatus

# Update definitions
Update-MpSignature

# Run quick scan
Start-MpScan -ScanType QuickScan

# Configure exclusions
Add-MpPreference -ExclusionPath "C:\MyApp"
Add-MpPreference -ExclusionExtension ".log"
```

---

*Windows administration combines traditional GUI tools with powerful command-line automation. Master PowerShell and you'll be able to manage Windows environments at any scale.*
