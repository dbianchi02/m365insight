---
title: "Configurare l’auto-rilevamento della posizione di lavoro in Microsoft Teams"
date: 2025-11-21T17:30:00+01:00
draft: false
tags: ["Governance", "Microsoft Teams", "Microsoft Places", "PowerShell"]
authors: ["Davide Bianchi"]
description: "Scopri come automatizzare la rilevazione della presenza in ufficio in Teams utilizzando SSID e BSSID della rete Wi-Fi aziendale, ottimizzando la gestione degli spazi."
---

## Introduzione: Gestire l'Ufficio Ibrido con Microsoft Places

In scenari di lavoro ibrido, sapere chi è in ufficio e dove si trova è diventato cruciale per una pianificazione efficace di riunioni e per l'ottimizzazione degli spazi aziendali. **Microsoft Places** consente di rilevare automaticamente la posizione di lavoro degli utenti in Teams quando si connettono alla rete Wi-Fi aziendale.

In questo articolo vedremo cos’è questo meccanismo, quali prerequisiti sono necessari e la procedura dettagliata in PowerShell per configurare il rilevamento tramite SSID e BSSID.

![WorkLocationPlace](/images/posts/governance/work-location.png)
---

### Cos’è il Rilevamento via Wi-Fi?

Quando un utente si collega a una rete Wi-Fi aziendale:

* Se è configurato solo l’SSID (Service Set Identifier), la posizione sarà **“In ufficio”**.
* Se sono configurati anche i BSSID (MAC address degli access point), la posizione sarà più precisa, permettendo l'associazione a un **edificio specifico**.

### Privacy e Consenso

La funzionalità rispetta la privacy dell'utente e non può essere forzata dall'amministratore:

* Gli utenti devono **accettare il rilevamento (opt-in)** direttamente in Teams.
* Gli amministratori non possono forzare il consenso.
* È necessario che la condivisione della posizione sia attiva nel sistema operativo e in Teams.

### Prerequisiti

Per iniziare, sono richiesti i seguenti ruoli e configurazioni:

#### Ruoli richiesti:
* Amministratore Teams per la gestione delle policy.
* Amministratore Exchange per la configurazione di Places.

#### Configurazioni necessarie:
* **Buildings** e **Floors** creati in Microsoft Places.

#### Strumenti:
* PowerShell con moduli `MicrosoftTeams` e `Places` installati e aggiornati.

---

### Configurazione Passo-Passo

#### 1. Abilitare la Policy di Rilevamento in Teams

Situazione iniziale in Teams prima di applicare la procedura:
![Before](/images/posts/governance/before-work-location.png)

Per prima cosa, crea la policy di rilevamento:

```powershell
# Creazione della Policy di Rilevamento della Posizione
New-CsTeamsWorkLocationDetectionPolicy -Identity wld-enabled -EnableWorkLocationDetection $true
```

Abilita la policy di rilevamento per un utente specifico/gruppo o globalmente:

```powershell
# Scegliere se abilitare la policy per specifici utenti o distribuirla globalmente
Grant-CsTeamsWorkLocationDetectionPolicy -PolicyName wld-enabled -Identity utente@dominio.com
```
#### 2. Configurare gli SSID (Posizione Generica)

Imposta gli SSID delle reti Wi-Fi aziendali che indicheranno genericamente la presenza "In ufficio":

```powershell
# Sostituisci 'SSID-1;SSID-2' con i nomi reali delle tue reti.
Set-PlacesSettings -Collection Presence -WorkplaceWifiNetworkSSIDList 'SSID-1;SSID-2' 
```
#### 3. Configurare BSSID per una Localizzazione Precisa (per Edificio)

Per associare una connessione a un edificio specifico, è necessario mappare i BSSID (MAC address degli Access Point) ai nomi degli edifici definiti in Places.

##### A. Preparare il File CSV di Mappatura

Prepara un file CSV (`mapping.csv`) con i seguenti dati:

| BSSID             | BuildingName            |
|-------------------|-------------------------|
| 00:0A:95:9D:68:16 | Sede Centrale Milano    |
| 00:1B:63:84:45:E6 | Filiale Roma            |

##### B. Creare la Mappatura

Utilizza il file CSV per creare la mappatura interna a Places.
```powershell
Add-WifiDevices -Action MapBuildings -InputFilePath mapping.csv
```
##### C. Caricare le Entry dei BSSID

Carica le entry dei BSSID associandole agli edifici in Places.

```powershell
Add-WifiDevices -Action UploadEntries -InputFilePath bssid.csv -BuildingMappingFile mapping.csv
```

_Suggerimento_: Se configuri solo SSID → posizione generica “In ufficio”. Se aggiungi BSSID → posizione dettagliata per edificio.

---

### Esperienza Utente

Dopo l’abilitazione, Teams chiederà all'utente il consenso.
La posizione si aggiorna automaticamente durante le working hours impostate in Outlook.
L'utente potrà sempre modificare l'opzione di condivisione della posizione accedendo a Teams -> Impostazioni -> Privacy
![After](/images/posts/governance/after-work-location.png)

---

### Conclusione

Configurare SSID e BSSID in Microsoft Places è il modo più efficace per automatizzare la rilevazione della presenza in ufficio, migliorando la collaborazione e la gestione degli spazi.

