# NMG Identity Automation 

PowerShell tooling for identity lifecycle management, built for
Northstar Medical Group.

## The Problem

Northstar's offboarding process lived in one person's box. 
When someone left the company, she'd email IT. That was it. 
That was the whole system.

Then she retired. Nobody replaced the process,just the person. 
It took 102 days before anyone realized accounts were sitting active 
with nobody behind them.

Someone eventually did it by hand. 23 stale accounts, 11 hours , spread across four days. 
Even then, it couldn't catch everything, no way to spot people whose
departure was never logged anywhere,, contractors who never showed up in a payroll,
or service accounts that were never tied to a person in the first place.

## The Approach

Stop trusting paperwork.Ask the domain controller instead.

Every account has a last authentication date. It doesn't care if
HR filed the right form or it two systems spell someone's name the same way.
It just knows when someone last logged in, of if they never did.

## Before you start

- Windows Server with the ActiveDirectory PowerShell module

      Import-Module ActiveDirectory

- Rights to modify user objects in the domain
- An authorising ticket number, in the form NMG-0000
- A Disabled Users OU at the root of the domain
- A writable reports folder. Create it if it does not exist:

      New-Item -Path "C:\Reports\Offboarding" -ItemType Directory -Force


## Tools

### Find-StaleAccounts.ps1

Flags enabled accounts that haven't authenticated in X days, plus accouts that
have never authenticated at all.Every run produces a timestampped CSV and a 
summary file, so the exact question behind the numbers never gets lost.
.\Find-StaleAccounts.ps1
    .\Find-StaleAccounts.ps1 -Days 30
    .\Find-StaleAccounts.ps1 -Days 180 -IncludeDisabled

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Days` | int | 90 | Days without authentication before an account is considered stale |
| `-ReportPath` | string | C:\Reports | Where reports are written |
| `-IncludeDisabled` | switch | off | Include disabled accounts in results |

**Output:** a timestamped CSV with findings, and a summary file that spells 
exactly what was asked to get them. Anyone can pick it up later and reproduce it.

### Offboard-NMGUser.ps1

Performs all five steps of SOP-IAM-001 against a single account.
Documents the account and its group memberships, verifies that
record on disk, disables the account, stamps the authorising
ticket, removes all group memberships, and moves the account to
the Disabled Users OU.

    .\Offboard-NMGUser.ps1 -Username "jdoe" -Ticket "NMG-0214" -WhatIf
    .\Offboard-NMGUser.ps1 -Username "jdoe" -Ticket "NMG-0214"

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Username` | string | required | SamAccountName of the account to offboard |
| `-Ticket` | string | required | Authorising ticket, in the form NMG-0000 |
| `-ReportPath` | string | C:\Reports\Offboarding | Where evidence files are written |
| `-LogPath` | string | Logs\ | Where the run transcript is written |
| `-WhatIf` | switch | off | Runs every check and changes nothing |

**Output:** two timestamped CSV files per account recording what
it was and what it could reach, plus a transcript of the run.

### Get-NMGOffboardingStatus.ps1

Reports how many accounts have been offboarded and quarantined,
how many are offboarded but not yet moved, and how many are still
waiting. Takes no parameters and makes no changes of any kind.

    .\Get-NMGOffboardingStatus.ps1

**Output:** three counts and two named lists, printed to the
console. Nothing is written to disk.


## When the script refuses

A refusal is the tool working correctly. It stops before making
any change at all, and tells you why.

| Message | What it means | What to do |
|---|---|---|
| no account named X | The username is wrong, or the account is already gone | Check the spelling in Active Directory |
| already disabled | Somebody has handled this one before you | Read the description field for the ticket number |
| looks like a service account | This is not a person | Find the owner. It needs a different procedure |
| ticket should look like NMG-0000 | The ticket format is wrong | Use the full four digit form |
| export file is empty | The record could not be written | Check the reports folder exists and is writable |
| Disabled Users OU not found | The destination is missing | Steps 1 to 4 completed. Move the account by hand |


## What an offboarding leaves behind

- Two timestamped CSV files per account in `Evidence/`. One
  records the account, one records every group it could reach.
- One transcript per run in `Logs/`, naming every membership
  removed.
- The authorising ticket number stamped on the account
  description in Active Directory.

The CSV of group memberships is the only record that will ever
exist of what an account could reach beforehand. Active Directory keeps no
history of a removed memberships.


## Known limitations

- Three accounts were offboarded before step 5 was implemented
  and were moved into the Disabled Users OU manually afterwards.
  Their logs do not record the move.
- Handles one account per run. Bulk processing is not built yet.
- The service account check matches on a name prefix and a
  department. An unusually named service account could get past it.

## Repository Structure

    Scripts/        PowerShell tools
    Documentation/  Runbooks and process documentation
    Evidence/       Sample output and verification screenshots
    Logs/           Execution logs

## Environment

Windows Server with Active Directory Domain Services.
Requires the ActiveDirectory PowerShell module.

## About

Built during the TotalThreat 30-Day Challenge in a simulated
healthcare environment. Northstar Medical Group is fictional.

Author: Fabella Terry
