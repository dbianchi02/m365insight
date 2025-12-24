---
title: "Microsoft Zero Trust Assessment: analizzare e migliorare la postura di sicurezza del tenant"
date: 2025-12-24T10:00:00+02:00
draft: false
tags: ["identità-sicurezza", "automazione-it", "Zero Trust", "Microsoft Graph", "PowerShell"]
authors: ["Davide Bianchi"]
description: "Guida pratica al Microsoft Zero Trust Assessment: prerequisiti, installazione del modulo PowerShell, esecuzione dell’analisi e interpretazione dei risultati per migliorare la postura Zero Trust del tenant."
---

## Introduzione

Il modello **Zero Trust** è diventato il riferimento architetturale per la sicurezza in ambienti cloud e ibridi: non esiste più una rete “fidata” per definizione, ma ogni accesso deve essere continuamente verificato, indipendentemente da posizione, utente o dispositivo.

Per supportare le organizzazioni in questo percorso, **:contentReference[oaicite:0]{index=0}** mette a disposizione il **Microsoft Zero Trust Assessment**, uno strumento ufficiale che consente di analizzare la configurazione di un tenant Microsoft 365 e valutarne l’allineamento ai principi Zero Trust.

In questo articolo vedremo:

* cos’è il Microsoft Zero Trust Assessment e a cosa serve;
* i prerequisiti necessari per eseguirlo;
* come installare ed eseguire l’assessment tramite PowerShell;
* come leggere e utilizzare correttamente i risultati finali.

![Overview](/images/posts/automazione/zta-overview.png)

---

## Cos’è il Microsoft Zero Trust Assessment

Il **Microsoft Zero Trust Assessment** è uno strumento di analisi **read-only**, distribuito come modulo PowerShell, che esegue una serie di controlli sulla configurazione del tenant interrogando **Microsoft Graph** e **Microsoft Azure**.

L’assessment verifica impostazioni legate principalmente ai pilastri:

* **Identity**
* **Devices**

e confronta lo stato attuale del tenant con le raccomandazioni Microsoft descritte nel modello Zero Trust.

Il risultato è un **report HTML locale**, dettagliato e navigabile, che evidenzia:
- controlli superati o falliti;
- livello di rischio associato;
- raccomandazioni di remediation per ogni test.

È importante chiarire fin da subito che:

> l’assessment **non applica modifiche automatiche** e **non “certifica” la sicurezza del tenant**, ma fornisce una fotografia tecnica dello stato attuale.

---

## A cosa serve il Microsoft Zero Trust Assessment

Lo strumento è pensato per supportare decisioni tecniche e architetturali, non per sostituirle.

In particolare, il Microsoft Zero Trust Assessment consente di:

* individuare gap di sicurezza nella configurazione del tenant;
* valutare il livello di maturità Zero Trust attuale;
* ottenere raccomandazioni tecniche concrete e contestualizzate;
* creare una baseline da cui partire per un piano di miglioramento progressivo;
* misurare nel tempo l’evoluzione della postura di sicurezza rieseguendo l’assessment.

Il valore principale non è il “punteggio”, ma la **qualità delle informazioni** che emergono dai singoli controlli.

---

## Prerequisiti

Prima di eseguire l’assessment è necessario verificare alcuni prerequisiti fondamentali.

### Requisiti tecnici

* **PowerShell 7** installato (Windows, Linux o macOS).
* Connettività verso Microsoft Graph e Microsoft Azure.

### Ruoli richiesti

* **Prima esecuzione**: account con ruolo **Global Administrator**, necessario per concedere i permessi all’app Microsoft Graph PowerShell.
* **Esecuzioni successive**: è sufficiente il ruolo **Global Reader**.

Se sul sistema è presente una versione precedente del modulo Zero Trust Assessment, questa deve essere **completamente rimossa** prima di procedere.

---

## Installazione del modulo Zero Trust Assessment

Apri una nuova sessione di **PowerShell 7** ed esegui:

```powershell
Install-Module ZeroTrustAssessment -Scope CurrentUser
```

Il modulo utilizza Microsoft Graph PowerShell e richiede, al primo utilizzo, il consenso a una serie di permessi read-only, necessari per analizzare identità, dispositivi, policy e configurazioni di sicurezza.

---

## Connessione a Microsoft Graph e Microsoft Azure

Per avviare l’assessment è necessario autenticarsi:

```powershell
Connect-ZtAssessment
```

Durante questa fase:

* viene richiesto il consenso ai permessi Microsoft Graph;
* si apre una seconda finestra di login per Microsoft Azure;
* se non è disponibile una sottoscrizione Azure, i test che dipendono dai log di Azure vengono semplicemente saltati.

Il modulo non modifica alcuna configurazione del tenant.

---

## Esecuzione dell’assessment

Una volta completata l’autenticazione, l’assessment si avvia con:

```powershell
Invoke-ZtAssessment
```

Caratteristiche principali dell’esecuzione:

* processo read-only;
* i dati vengono salvati localmente;
* su tenant di grandi dimensioni l’esecuzione può richiedere diverse ore (in alcuni casi oltre 24 ore);
* l’assessment non deve essere interrotto durante l’esecuzione.

Al termine viene generato il report: *.\ZeroTrustReport\ZeroTrustAssessmentReport.html*

Il file si apre automaticamente nel browser predefinito.

> ⚠️ Il report contiene informazioni sensibili sul tenant: deve essere conservato e condiviso solo con personale autorizzato.

È possibile specificare un percorso personalizzato tramite il parametro -Path.

---

## Analisi dei risultati

Il report è organizzato in più sezioni.

La scheda Overview fornisce una vista di alto livello sullo stato Zero Trust del tenant.
Le sezioni Identity e Devices mostrano l’elenco dettagliato dei test eseguiti, con:

* stato del controllo (Pass / Fail / Warning);
* livello di rischio;
* descrizione tecnica del test;
* raccomandazioni di remediation.

![Overview](/images/posts/automazione/results-identity.png)

Selezionando un singolo controllo è possibile capire:

* cosa è stato verificato;
* perché è rilevante in ottica Zero Trust;
* quali azioni correttive sono suggerite.

![Overview](/images/posts/automazione/sample-test.png)

---

## Come applicare i risultati finali

Un errore comune è interpretare l’assessment come una checklist da completare al 100%.
In realtà, i risultati vanno contestualizzati rispetto a:

* dimensione dell’organizzazione;
* requisiti di business;
* vincoli operativi e di licensing;
* modello di rischio accettabile.

Un approccio efficace consiste nel:

* partire dalle raccomandazioni ad alto impatto e bassa complessità;
* integrare le remediation in un piano di sicurezza strutturato;
* documentare le scelte consapevoli di non applicare alcune raccomandazioni;
* **rieseguire periodicamente** l’assessment per misurare i progressi.

Zero Trust è un **percorso continuo**, non uno stato finale.

---

## Conclusione

Il Microsoft Zero Trust Assessment è uno strumento estremamente utile per chi vuole affrontare il tema Zero Trust in modo concreto, misurabile e allineato alle best practice Microsoft.
Se utilizzato correttamente, diventa una guida tecnica affidabile per migliorare progressivamente la postura di sicurezza del tenant Microsoft 365, supportando decisioni architetturali e di governance basate su dati reali.