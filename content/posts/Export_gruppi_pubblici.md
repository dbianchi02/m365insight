---
title: "Identificare e gestire i gruppi M365 pubblici: script PowerShell per la governance"
date: 2025-12-28T10:00:00+02:00
draft: false
tags: ["automazione-it", "Microsoft Graph", "PowerShell", "Microsoft 365"]
authors: ["Davide Bianchi"]
description: "Script PowerShell per identificare e monitorare i gruppi Microsoft 365 pubblici, riducendo i rischi di oversharing e migliorando la governance del tenant."
---

## Introduzione

Uno dei problemi più sottovalutati nella gestione di un tenant Microsoft 365 è la presenza di **gruppi pubblici creati involontariamente** dagli utenti. Questi gruppi rappresentano un rischio significativo per la sicurezza dei dati, specialmente con l'introduzione di Microsoft 365 Copilot, che ha accesso a tutti i contenuti visibili all'utente, inclusi quelli dei gruppi pubblici di cui l'utente potrebbe non essere nemmeno consapevole.

In questo articolo vi mostro uno script PowerShell che utilizzo regolarmente con i miei clienti per identificare tutti i gruppi M365 pubblici presenti nel tenant, estraendo informazioni dettagliate su ciascuno di essi. Questo script rappresenta un punto di partenza fondamentale per qualsiasi strategia di governance efficace.

---

## Il problema dei gruppi pubblici

Prima di analizzare lo script, è importante capire perché i gruppi pubblici rappresentano un problema.

Quando un utente crea un gruppo M365 (tramite Teams, Outlook, Planner o altri workload), ha la possibilità di scegliere tra due livelli di visibilità:

- **Privato**: solo i membri possono vedere i contenuti del gruppo  
- **Pubblico**: tutti gli utenti dell'organizzazione possono vedere e accedere ai contenuti  

Il problema nasce quando gli utenti:

1. Non comprendono la differenza tra pubblico e privato  
2. Selezionano "pubblico" pensando significhi semplicemente "interno all'azienda"  
3. Non si rendono conto di rendere accessibili documenti potenzialmente sensibili a tutta l'organizzazione  

Con l'arrivo di Copilot, questo problema si amplifica ulteriormente: anche se un utente non ha mai cercato o scoperto l'esistenza di un gruppo pubblico, **Copilot sa che l'utente ha accesso a quei contenuti e li utilizzerà nelle sue risposte**.

---

## Lo script di estrazione

Lo script presentato in questo articolo consente di estrarre una lista completa di tutti i gruppi pubblici presenti nel tenant, includendo informazioni dettagliate quali:

- Nome e descrizione del gruppo  
- Data di creazione  
- Owner del gruppo (fondamentale per identificare i responsabili)  
- Membri del gruppo  
- URL del sito SharePoint associato (se presente)  

Lo script completo è riportato di seguito. Salvatelo come file `.ps1` (ad esempio `Export-PublicGroups.ps1`) per poterlo eseguire.

```powershell
Connect-MgGraph
Connect-ExchangeOnline

# Retrieve all public groups
$groups = Get-MgGroup -All | ?{$_.Visibility -eq "Public"} 
$date = Get-Date -Format dd-MM-yyyy
$csvFilePath = "C:\temp\M365_Groups_Public_$date.csv"

$results = @()

foreach ($group in $groups) {
    $groupInfo = [PSCustomObject]@{
        'Group Name' = $group.DisplayName
        'Group ID' = $group.Id
        'MailNickName' = $group.mailnickname
        'Description' = $group.Description
        'CreatedDateTime' = $group.CreatedDateTime
        'Visibility' = $group.Visibility
        'Group Type' = $group.GroupTypes -join ', '
    }

    # Retrieve group members
    $members = Get-MgGroupMember -GroupId $group.Id
    $memberNames = @()

    foreach ($member in $members) {

        $memberDetails = Get-MgUser -UserId $member.Id
        $memberNames += $memberDetails.DisplayName

    }
    $groupInfo | Add-Member -MemberType NoteProperty -Name 'Members' -Value ($memberNames -join ', ')

    # Retrieve group owners
    $owners = Get-MgGroupOwner -GroupId $group.Id
    $ownerNames = @()

    foreach ($owner in $owners) {

        $ownerDetails = Get-MgUser -UserId $owner.Id
        $ownerNames += $ownerDetails.DisplayName

    }
    $groupInfo | Add-Member -MemberType NoteProperty -Name 'Owners' -Value ($ownerNames -join ', ')

    #Get information about the Microsoft 365 Group
    $groupSPcheck = Get-UnifiedGroup -Identity $group.id
    $SharePointSiteURL = @()

    #Check if the group has a SharePoint site associated with it
    if ($groupSPcheck.SharePointSiteUrl) {

        $SharePointSiteURL = $($groupSPcheck.SharePointSiteUrl) -join "`n"

    } else {

        $SharePointSiteURL = "No SharePoint Site"

    }
    $groupInfo | Add-Member -MemberType NoteProperty -Name 'SharePointSiteURL' -Value ($SharePointSiteURL -join ', ')




    $results += $groupInfo
}

# Export results to CSV
$results | Export-Csv -Path $csvFilePath -NoTypeInformation -Delimiter ";"

Write-Host "Export complete. Results saved to: $csvFilePath"
```

---

## Prerequisiti

Prima di eseguire lo script, assicuratevi di soddisfare i seguenti prerequisiti.

### Moduli PowerShell installati

- Microsoft Graph PowerShell SDK  
- Exchange Online Management Module  

### Permessi richiesti

- Lettura di gruppi e utenti in Azure AD  
- Accesso in lettura alle configurazioni di Exchange Online  

---

## Come utilizzare lo script

### 1. Analisi dei risultati

Aprite il file CSV generato utilizzando Excel. Il file conterrà una riga per ogni gruppo pubblico, con tutte le informazioni estratte.

#### Cosa analizzare

**Gruppi senza owner o con owner inattivi**  
Questi gruppi sono da considerarsi “orfani” e rappresentano il rischio più elevato. Devono essere riassegnati o eliminati.

**Gruppi con siti SharePoint associati**  
È consigliabile verificare manualmente i contenuti di questi siti per valutare la sensibilità dei dati esposti.

**Gruppi creati recentemente**  
Consentono di individuare trend: un aumento della creazione di gruppi pubblici può indicare la necessità di formazione o di misure preventive.

**Gruppi con pochi o nessun membro**  
Potrebbero essere gruppi obsoleti, candidati all’eliminazione.

---

### 2. Azioni di remediation

Una volta identificati i gruppi problematici, è possibile procedere con le seguenti azioni.

**Contattare gli owner**  
Spiegate i rischi legati ai gruppi pubblici e valutate insieme se convertire il gruppo in privato o se la visibilità pubblica è effettivamente necessaria.

**Convertire i gruppi in privato**  
Per i gruppi che non devono essere pubblici, utilizzate i comandi PowerShell appropriati per modificare la visibilità.

**Rimuovere gruppi obsoleti**  
Per gruppi non più utilizzati o privi di owner attivi, valutate l’eliminazione dopo aver archiviato eventuali contenuti rilevanti.

**Documentare le decisioni**  
Tenete traccia delle analisi effettuate, delle azioni intraprese e delle motivazioni che giustificano la permanenza di gruppi pubblici legittimi.

---

### 3. Monitoraggio continuo

Il vero valore di questo script emerge con l’**esecuzione periodica**.

#### Buone pratiche consigliate

**Schedulare esecuzioni settimanali**  
Utilizzate Task Scheduler o Azure Automation per automatizzare l’esecuzione.

**Confrontare i risultati nel tempo**  
Grazie al nome del file che include la data, è possibile confrontare facilmente le estrazioni e identificare:

- nuovi gruppi pubblici creati  
- gruppi corretti dopo le azioni di remediation  
- trend di miglioramento o peggioramento  

**Creare metriche di governance**

Monitorate nel tempo:

- numero totale di gruppi pubblici  
- percentuale di gruppi pubblici rispetto al totale  
- tempo medio di remediation  
- numero di nuovi gruppi pubblici creati per settimana o mese  

---

### 4. Integrazione con la strategia di governance

Questo script rappresenta solo il primo passo.

Per una governance realmente efficace è consigliabile affiancarlo a:

**Prevenzione**  
Come descritto nel mio articolo ➡️ [Controllare i gruppi M365 con Sensitivity Labels](https://m365insight.dbianchi.it/posts/block_public_teams/), l’utilizzo delle **sensitivity labels** consente di bloccare la creazione di nuovi gruppi pubblici, permettendo solo gruppi privati.

**Formazione**  
I dati raccolti aiutano a identificare dipartimenti o utenti che creano frequentemente gruppi pubblici, consentendo di pianificare sessioni di formazione mirate.

**Policy e attestazione**  
Integrare questo controllo con processi di attestazione periodica in cui gli owner confermano che la visibilità del gruppo è ancora appropriata.

**Reporting**  
Creare report mensili per il management per mostrare l’andamento della situazione e l’efficacia delle azioni intraprese.

---

## Conclusioni

I gruppi M365 pubblici sono uno di quei problemi che restano invisibili finché non diventano critici. Con l’introduzione di Copilot, ciò che prima era un rischio teorico diventa un rischio concreto e immediato.

Questo script fornisce la visibilità necessaria per identificare e correggere rapidamente il problema. Il consiglio è semplice: eseguitelo oggi stesso, analizzate i risultati e iniziate a collaborare con gli owner per sistemare le configurazioni non corrette.

La governance dei dati non è un’attività una tantum, ma un processo continuo. Questo script è uno degli strumenti fondamentali da avere nella vostra cassetta degli attrezzi per mantenere il controllo del vostro ambiente Microsoft 365.
