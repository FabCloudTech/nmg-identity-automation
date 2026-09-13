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

Author: [Fabella Terry]
