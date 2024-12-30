# azure-defender-asbuilt-environmnet-settings
Script to download azure defender environment settings 

Certainly! To add logging and verbose output to the script for downloading Azure Defender subscription configuration environment settings, we can use additional PowerShell features to provide more detailed information about each step of the process. Here is the enhanced script with logging and verbose output:

```powershell
# Enable verbose output
$VerbosePreference = "Continue"

# List of subscription IDs
$subscriptionIds = @(
    "subscription-id-1",
    "subscription-id-2",
    "subscription-id-3"
)

# Directory to save the configuration files
$outputDirectory = "C:\AzureDefenderConfigs"
if (-Not (Test-Path $outputDirectory)) {
    New-Item -ItemType Directory -Path $outputDirectory -Verbose
}

# Log file path
$logFilePath = Join-Path $outputDirectory "DownloadLog.txt"

# Function to log messages
function Log-Message {
    param (
        [string]$message
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "$timestamp - $message"
    Write-Output $logMessage
    Add-Content -Path $logFilePath -Value $logMessage
}

Log-Message "Script execution started."

foreach ($subscriptionId in $subscriptionIds) {
    try {
        Log-Message "Processing subscription ID: $subscriptionId"

        # Set the current subscription
        az account set --subscription $subscriptionId
        Log-Message "Set current subscription to $subscriptionId"

        # Get the Azure Defender configuration for the current subscription
        $defenderConfig = az security pricing list --query "[?pricingTier=='Standard']" -o json
        Log-Message "Retrieved Azure Defender configuration for $subscriptionId"

        # Convert the configuration to JSON
        $configJson = $defenderConfig | ConvertTo-Json -Depth 10
        Log-Message "Converted configuration to JSON for $subscriptionId"

        # Save the JSON to a file
        $outputFile = Join-Path $outputDirectory "$subscriptionId-DefenderConfig.json"
        Set-Content -Path $outputFile -Value $configJson -Verbose
        Log-Message "Configuration saved to $outputFile for subscription $subscriptionId"

    } catch {
        Log-Message "Error processing subscription $subscriptionId: $_"
    }
}

Log-Message "Script execution completed."
Write-Output "All configurations have been downloaded and saved."
```

### Explanation
1. **Verbose Output**: The `$VerbosePreference` variable is set to "Continue" to enable verbose output for all commands that support it.
2. **Log Function**: A `Log-Message` function is defined to write log messages to a log file with a timestamp.
3. **Log File Path**: The path for the log file is specified, and the log file is created in the output directory.
4. **Logging Steps**: Log messages are added at key points in the script to provide information about the current operation and any errors encountered.
5. **Try-Catch Block**: A `try-catch` block is used to handle any errors that occur during the processing of each subscription, and errors are logged accordingly.

### Running the Script
1. Open a PowerShell window.
2. Run the script.

This script will now provide detailed logging information in the console and save the log messages to a log file in the specified output directory. This will help you track the progress and debug any issues that may arise.
