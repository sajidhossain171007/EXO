# Prompt for the email address of the target mailbox
$emailAddress = Read-Host "Enter the email address of the target mailbox"

# Import required modules
Import-Module ExchangeOnlineManagement

# Prompt for admin credentials
$credentials = Get-Credential -Message "Enter your admin credentials"

# Connect to Exchange Online and Security & Compliance PowerShell
Connect-ExchangeOnline -Credential $credentials -ShowBanner:$false -CommandName Get-MailboxFolderStatistics
Connect-IPPSSession -Credential $credentials -ShowBanner:$false

# Retrieve folder statistics for the target mailbox
$folderStatistics = Get-MailboxFolderStatistics $emailAddress
$recoverableItemsFolderQuery = ""

# Iterate through folder statistics to locate the 'Recoverable Items' folder
foreach ($folderStatistic in $folderStatistics) {
    $folderPath = $folderStatistic.FolderPath
    if ($folderPath -eq "/Recoverable Items") {
        $folderId = $folderStatistic.FolderId
        $encoding = [System.Text.Encoding]::GetEncoding("us-ascii")
        $nibbler = $encoding.GetBytes("0123456789ABCDEF")
        $folderIdBytes = [Convert]::FromBase64String($folderId)
        $indexIdBytes = New-Object byte[] 48
        $indexIdIdx = 0
        $folderIdBytes | Select-Object -Skip 23 -First 24 | ForEach-Object {
            $indexIdBytes[$indexIdIdx++] = $nibbler[$_ -shr 4]
            $indexIdBytes[$indexIdIdx++] = $nibbler[$_ -band 0xF]
        }

        $recoverableItemsFolderQuery = "folderid:$($encoding.GetString($indexIdBytes))"
        break
    }
}

# If the recoverable items folder query is not empty, create a compliance search
if ($recoverableItemsFolderQuery -ne "") {
    $searchName = "RecoverableItemsSearch_$emailAddress"
    $description = "Compliance search for '/Recoverable Items' folder of mailbox '$emailAddress'"
    New-ComplianceSearch -Name $searchName -ContentMatchQuery $recoverableItemsFolderQuery -Description $description -ExchangeLocation $emailAddress -Force
    Write-Host "Compliance search created with name '$searchName', description '$description', and Exchange location '$emailAddress'."
} else {
    Write-Host "No '/Recoverable Items' folder found in the specified mailbox."
}

# Initiate the compliance search
Start-ComplianceSearch -Identity $searchName
