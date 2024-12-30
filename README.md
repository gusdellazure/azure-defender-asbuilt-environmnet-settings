# Enable verbose output
$VerbosePreference = "Continue"

# List of subscription IDs
$subscriptionIds = @(
    "subscription-id-1",
    "subscription-id-2",
    "subscription-id-3"
)

# Directory to save the configuration files (same location as the script file)
$outputDirectory = $PSScriptRoot
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

        # Check if the output directory and file path are set correctly
        if (-Not $outputDirectory) {
            throw "Output directory is not set."
        }
        if (-Not $configJson) {
            throw "Configuration JSON is null or empty."
        }

        # Save the JSON to a file
        $outputFile = Join-Path $outputDirectory "$subscriptionId-DefenderConfig.json"
        if (-Not $outputFile) {
            throw "Output file path is not set."
        }
        Set-Content -Path $outputFile -Value $configJson -Verbose
        Log-Message "Configuration saved to $outputFile for subscription $subscriptionId"

    } catch {
        Log-Message "Error processing subscription $subscriptionId: $($_.Exception.Message)"
    }
}

Log-Message "Script execution completed."
Write-Output "All configurations have been downloaded and saved."
