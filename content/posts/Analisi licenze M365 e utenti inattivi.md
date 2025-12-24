---
title: "Analisi licenze Microsoft 365: individuare utenti inattivi e licenze sprecate"
date: 2025-12-24T10:00:00+02:00
draft: false
tags: ["automazione-it", "Inactive-Report", "Microsoft Graph", "PowerShell"]
authors: ["Davide Bianchi"]
description: "Uno script PowerShell che utilizzo nei tenant dei clienti per individuare utenti inattivi, account disabilitati con licenze assegnate e opportunità concrete di ottimizzazione delle licenze Microsoft 365."
---

## Introduzione

La gestione delle licenze Microsoft 365 è uno degli aspetti più sottovalutati nei tenant aziendali.  
Nella maggior parte dei clienti che seguo, il problema non è la mancanza di funzionalità, ma **la dispersione delle licenze**: utenti disabilitati ancora licenziati, account che non effettuano accessi da mesi e licenze che continuano a generare costi senza un reale utilizzo.

Per questo motivo, nel tempo ho sviluppato uno **script di analisi** che utilizzo regolarmente durante assessment, attività di governance o fasi di ottimizzazione dei costi.  
L’obiettivo non è solo “fare pulizia”, ma **fornire dati oggettivi** su cui basare decisioni consapevoli.

![Overview](/images/posts/automazione/license-opt.png)

---

## Il problema: licenze assegnate senza reale utilizzo

Nei tenant Microsoft 365 maturi, soprattutto quelli cresciuti nel tempo, è frequente trovare situazioni come:

- utenti **disabilitati** che mantengono una o più licenze attive;
- account che **non effettuano sign-in da mesi**, ma risultano ancora licenziati;
- utenti tecnici o legacy mai realmente utilizzati;
- assenza di una visione centralizzata sull’inattività degli account.

Queste situazioni hanno due impatti principali:
1. **economico**, perché le licenze continuano a essere fatturate;
2. **di sicurezza**, perché account inutilizzati aumentano la superficie di attacco.

---

## Cosa fa lo script (a livello funzionale)

Lo script nasce per rispondere a una domanda molto semplice:

> *“Quante licenze sto pagando senza che vengano realmente utilizzate?”*

Dal punto di vista funzionale, lo script esegue tre analisi distinte sul tenant Microsoft 365:

1. **Utenti disabilitati con licenze assegnate**  
   Account non più attivi ma che continuano a consumare licenze.

2. **Utenti con licenze assegnate inattivi da oltre 100 giorni**  
   Utenti che non effettuano accessi da lungo tempo, pur avendo una o più licenze attive.

3. **Utenti inattivi da oltre 100 giorni (indipendentemente dalle licenze)**  
   Una vista più ampia sull’inattività, utile anche per valutazioni di sicurezza e governance.

I risultati vengono esportati in file CSV separati, pronti per essere analizzati o condivisi con il cliente.

---

## Perché uso Microsoft Graph

Lo script si basa su **Microsoft Graph**, sfruttando in particolare:

- informazioni sugli utenti del tenant;
- stato di abilitazione degli account;
- assegnazione delle licenze;
- attività di accesso (sign-in).

Questo approccio consente di:
- lavorare in modalità **read-only**;
- evitare dipendenze da moduli legacy;
- utilizzare un modello coerente con le best practice Microsoft.

Le autorizzazioni richieste sono limitate alla lettura dei dati necessari per l’analisi.

---

## Struttura dell’output e modalità di utilizzo

All’avvio, lo script chiede di specificare il **nome del cliente** e crea una cartella dedicata all’export dei risultati.  
Questo approccio è particolarmente utile quando si lavora su più tenant o in contesti di consulenza.

Ogni esecuzione genera:
- file CSV separati per ciascun tipo di analisi;
- dati facilmente filtrabili, aggregabili e confrontabili;
- una base oggettiva per discutere azioni correttive con il cliente.

I file possono essere:
- analizzati manualmente;
- importati in Excel o Power BI;
- utilizzati come input per ulteriori automazioni.

---

## Casi d’uso reali

Questo script lo utilizzo tipicamente in tre contesti principali.

### Assessment iniziale del tenant

Durante le prime fasi di presa in carico di un cliente, lo script permette di:
- individuare rapidamente sprechi evidenti;
- stimare un potenziale risparmio economico;
- fornire valore immediato, basato su dati concreti.

### Ottimizzazione periodica delle licenze

In tenant di medie e grandi dimensioni, lo script può essere eseguito periodicamente per:
- monitorare l’inattività degli account;
- supportare processi di deprovisioning;
- allineare l’assegnazione delle licenze all’utilizzo reale.

### Supporto a decisioni di governance e sicurezza

Gli utenti inattivi non sono solo un tema di costi.  
I risultati dello script sono spesso un ottimo punto di partenza per:
- rivedere processi di joiner/mover/leaver;
- introdurre policy di revisione periodica degli account;
- ridurre il rischio legato ad account inutilizzati.

---

## Limiti e considerazioni importanti

È importante interpretare correttamente i risultati.

Un utente inattivo da 100 giorni **non è automaticamente da rimuovere**.  
Potrebbe trattarsi, ad esempio, di:
- account stagionali;
- utenti in aspettativa;
- account tecnici con accessi sporadici.

Lo script **non prende decisioni**, ma fornisce dati.  
La responsabilità finale resta sempre del team IT o del cliente.

---

## Script PowerShell

```Powershell
<# 
********************************************************************************************************************
#SCRIPT: ANALISI LICENZE & UTENTI INATTIVI 
INFO

    Author          :    Bianchi Davide
    mail            :    hello@dbianchi.it
    Company         :    M365 Insight (https://m365insight.dbianchi.it)
    Current Version :    0.1
    Created         :    11/10/2024
    Last Updated    :    11/10/2024
    Request         :    Get Inactive Users information

********************************************************************************************************************
DESCRIPTION

    Questo script estrae dati di analisi utenze per un tenant Office 365, tra cui utenti disabilitati con licenze e 
    utenti inattivi da oltre 100 giorni. All'avvio, crea una cartella personalizzata dedicata al cliente, in cui 
    salva i risultati in formato CSV. Lo script richiede l'accesso a Microsoft Graph, utilizzando le autorizzazioni 
    necessarie per audit e lettura degli utenti del tenant.
   
    Permission Microsoft.Graph: AuditLog.Read.All & User.Read.All

********************************************************************************************************************
VERSION

    0.1 | Prima bozza dello script, nessuna implementazione

********************************************************************************************************************
#>

#*******************************************************************************************************************#
#---------------------------------------------------- FUNZIONI -----------------------------------------------------#
#*******************************************************************************************************************#

function Write-Success {
    param ([string]$message)
    Write-Host $message -ForegroundColor Green
}

#*******************************************************************************************************************#
#---------------------------------------------------- VARIABILI ----------------------------------------------------#
#*******************************************************************************************************************#

#Prendo la data di oggi per ottenere un file diverso ogni volta che viene lanciato lo script nella stessa posizione
$cliente = Read-Host -Prompt "Dichiarare nome cliente"
$OutputFolderPath = "C:\Temp\Office365Assessment_$cliente"

# Controlla se la cartella esiste già, altrimenti la crea
if (-Not (Test-Path -Path $OutputFolderPath)) {
    New-Item -ItemType Directory -Path $OutputFolderPath
    Write-Host "Cartella creata: $OutputFolderPath"
} else {
    Write-Host "La cartella $OutputFolderPath esiste già."
}

#Dichiarazione export file che verranno salvati nella cartella 

$LicensedDisabledUsersEx = "$OutputFolderPath\Licensed_Disabled_Users.csv"
$LicensedInactive100daysUsersEx = "$OutputFolderPath\Licensed_Inactive_100days_Users.csv"
$Inactive100daysUsersEx = "$OutputFolderPath\Inactive_100days_Users.csv"

#*******************************************************************************************************************#
#------------------------------------------- MICROSOFT GRAPH CONNECTION --------------------------------------------#
#*******************************************************************************************************************#

#Connessione a Microsoft Graph
$ClientID = "Inserire App ID del cliente"
$TenantID = "Inserire Tenant ID del cliente"
$ClientSecret = "Inserire Client Secret del cliente"

$body = @{
    grant_type    = "client_credentials"
    scope         = "https://graph.microsoft.com/.default"
    client_id     = $ClientID
    client_secret = $ClientSecret
}

$oauth = Invoke-RestMethod -Method Post -Uri "https://login.microsoftonline.com/$($TenantID)/oauth2/v2.0/token" -Body $body
$token = @{ 'Authorization' = "$($oauth.token_type) $($oauth.access_token)" }

#New row to fix error in connection to MgGraph version 2.6.1
#Convert token to a secure string
$securetoken = ConvertTo-SecureString "$($oauth.access_token)" -AsPlainText -Force

#Connect to MgGraph using oauth2 token
Connect-MgGraph -AccessToken $securetoken

#*******************************************************************************************************************#
#------------------------------------------ EXPORT LICENSED DISABLED USERS -----------------------------------------#
#*******************************************************************************************************************#

# Ottieni tutti gli utenti disabilitati
$disabledUsers = Get-MgUser -Filter "accountEnabled eq false" -ConsistencyLevel eventual -All -Select UserPrincipalName,DisplayName,AssignedLicenses

# Filtra gli utenti disabilitati per quelli che hanno licenze assegnate
$disabledUsersWithLicenses = $disabledUsers | Where-Object { $_.AssignedLicenses.Count -gt 0 }

# Visualizza i risultati
$disabledUsersWithLicenses | Select-Object UserPrincipalName, DisplayName, AssignedLicenses | Export-Csv -Path $LicensedDisabledUsersEx -Delimiter ';'-NoTypeInformation
Write-Success "Utenti disabilitati esportati in $OutputFolderPath"

#*******************************************************************************************************************#
#-------------------------------------- EXPORT LICENSED INACTIVE 100 DAYS USERS ------------------------------------#
#*******************************************************************************************************************#

# Prendo tutti gli utenti del tenant 
$AllUser = Get-MgUser -Property 'SignInActivity,AssignedLicenses' -All | select DisplayName, UserPrincipalName, AssignedLicenses | fl

# Dichiaro un array vuoto per l'object report
$noLoginWithLicensesInfo = @()

# Dichiaro la data di oggi -100 giorni per fare il confronto con il last sign in date
$today = (Get-date).AddDays(-100)

# Inizio il ciclo foreach e ottengo la last logon date e le licenze per ogni utente
foreach ($user in $AllUser) {
    $lastSignInDate = $user.SignInActivity.LastSignInDateTime
    $assignedLicenses = $user.AssignedLicenses

    # Verifico se l'utente non ha fatto login da più di 100 giorni e ha almeno una licenza assegnata
    if ($assignedLicenses.Count -gt 0 -and ($null -eq $lastSignInDate -or $lastSignInDate -lt $today)) {
        $noLoginWithLicensesInfo += [PSCustomObject]@{
            DisplayName = $user.DisplayName
            UserPrincipalName = $user.UserPrincipalName
            LastSignInDate = if ($null -eq $lastSignInDate) { "Nessun sign-in trovato" } else { $lastSignInDate }
            AssignedLicenses = $assignedLicenses.Count
        }
    }
}

# Esporto tutti i risultati ottenuti in un CSV, e scrivo nella cmdlet che lo script è terminato e dove è stato esportato il file
$noLoginWithLicensesInfo | Export-Csv -Path $LicensedInactive100daysUsersEx -Delimiter ';' -NoTypeInformation 
Write-Success "Utenti inattivi esportati in $OutputFolderPath"

#*******************************************************************************************************************#
#------------------------------------------ EXPORT INACTIVE 100 DAYS USERS -----------------------------------------#
#*******************************************************************************************************************#

# Prendo tutti gli utenti del tenant 
$AllUser = Get-MgUser -Property 'SignInActivity' -All | select DisplayName, UserPrincipalName, SignInActivity

# Dichiaro un array vuoto per l'object report
$noLoginInfo = @()

# Dichiaro la data di oggi -100 giorni per fare il confronto con la last sign in date
$today = (Get-date).AddDays(-100)

# Inizio il ciclo foreach e ottengo la last logon date per ogni utente
foreach ($user in $AllUser) {
    $lastSignInDate = $user.SignInActivity.LastSignInDateTime

    # Verifico se l'utente non ha fatto login da più di 100 giorni
    if ($null -eq $lastSignInDate -or $lastSignInDate -lt $today) {
        $noLoginInfo += [PSCustomObject]@{
            DisplayName = $user.DisplayName
            UserPrincipalName = $user.UserPrincipalName
            LastSignInDate = if ($null -eq $lastSignInDate) { "Nessun sign-in trovato" } else { $lastSignInDate }
        }
    }
}

# Esporto tutti i risultati ottenuti in un CSV e scrivo nella cmdlet che lo script è terminato
$noLoginInfo | Export-Csv -Path $Inactive100daysUsersEx -Delimiter ';' -NoTypeInformation 
Write-Success "Utenti inattivi esportati in $OutputFolderPath"

#*******************************************************************************************************************#
#------------------------------------------------------ FINE SCRIPT ------------------------------------------------#
#*******************************************************************************************************************#

Write-Host "Risultati esportati in $OutputFolderPath" -Foregroundcolor Green 
Write-Success "Tutti i file di assessment sono pronti per le analisi. Lo script è terminato!"

```

---

## Conclusione

Una gestione efficace delle licenze Microsoft 365 non passa solo dal portale di amministrazione, ma da **dati reali sull’utilizzo**.  
Questo script nasce proprio con questo obiettivo: trasformare informazioni tecniche in decisioni concrete, riducendo sprechi e migliorando la governance del tenant.

Non è uno strumento “magico”, ma una base solida su cui costruire processi più maturi di gestione delle identità e delle licenze in Microsoft 365.