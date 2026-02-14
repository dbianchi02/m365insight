---
title: "Blocco della creazione di Gruppi M365 Pubblici in Microsoft Teams e SharePoint Online tramite Sensitivity Labels"
date: 2026-02-14T10:00:00+02:00
draft: false
tags: ["Governance", "Microsoft Teams", "Sensitivity Labels", "Sicurezza M365", "Microsoft Purview"]
authors: ["Davide Bianchi"]
description: "Scopri come impedire la creazione di Gruppi Microsoft 365 Pubblici in Microsoft Teams utilizzando le Sensitivity Labels di Microsoft Purview, garantendo maggiore controllo e sicurezza."
---

## Introduzione

In un ambiente collaborativo come Microsoft Teams, la facilità con cui gli utenti possono creare nuovi spazi è al tempo stesso un punto di forza e una potenziale fonte di rischio. La proliferazione incontrollata di gruppi M365 pubblici può esporre dati sensibili, creare confusione organizzativa e rendere difficile mantenere una governance coerente del tenant.

La soluzione più efficace non è limitare la creazione di gruppi in modo generalizzato, ma imporre una configurazione di privacy controllata fin dal momento della creazione. Questo articolo mostra come farlo attraverso le **Sensitivity Labels** di **Microsoft Purview Information Protection**.

---

## Perché le Sensitivity Labels sono la scelta giusta

In Microsoft Teams, creare un team significa creare un Gruppo Microsoft 365. Se l'obiettivo è impedire la nascita di gruppi pubblici mantenendo però la possibilità di creare team privati, la strada corretta non è bloccare la creazione tout court, ma governarla.

Le Sensitivity Labels permettono agli amministratori di definire regole di privacy applicate automaticamente durante la creazione di gruppi, team e siti SharePoint. Il risultato è un controllo granulare che non ostacola la produttività degli utenti, ma la incanalizza all'interno dei confini definiti dall'organizzazione.

---

## Prerequisiti

Prima di procedere, assicurati di avere:

- Una **licenza Microsoft 365** che supporti le Sensitivity Labels per Teams, SharePoint e Gruppi (tipicamente M365 E3 con add-on o E5, o equivalenti).
- Accesso al **Microsoft Purview Compliance Portal** (`https://compliance.microsoft.com`).
- Permessi di amministratore per creare e pubblicare etichette.

---

## Procedura passo-passo

### 1. Abilitare le Sensitivity Labels per Teams e Gruppi

Come prima cosa, verifica che la funzionalità sia attiva nel tuo tenant.

Accedi al **Microsoft Purview Compliance Portal** → **Information Protection** → **Labels** e controlla che le etichette siano abilitate per **Teams, Groups e Sites**.

![Purview Portal](/images/posts/governance/label-portal.png)

> Se la funzionalità non risulta attiva, puoi abilitarla via PowerShell seguendo la [guida ufficiale Microsoft](https://learn.microsoft.com/en-us/entra/identity/users/groups-assign-sensitivity-labels?tabs=microsoft#enable-sensitivity-label-support-in-powershell).

---

### 2. Creare la Sensitivity Label

Ora creiamo l'etichetta che imporrà la privacy al momento della creazione del gruppo. Clicca su **Create a label** e configura i parametri in modo che siano chiari sia per gli amministratori che per gli utenti finali. Di seguito la configurazione che ho scelto:

| Campo | Valore |
|---|---|
| **Name** | Block Public Teams |
| **Display Name** | Collaborazione interna – Gruppi privati |
| **Description for users** | Usa questa etichetta per creare Team, Gruppi e Siti SharePoint visibili solo ai membri. L'organizzazione non consente la creazione di gruppi pubblici. |
| **Description for admins** | Questa etichetta imposta la privacy dei Gruppi Microsoft 365, Team e siti SharePoint su Privato, impedendo la creazione di gruppi pubblici. |

Per lo **Scope**, seleziona **Groups & Sites**: questo include automaticamente Teams e i Gruppi M365.

Nelle impostazioni di protezione (**Protection Settings**), seleziona **Privacy and external user access** e imposta **Privacy** su **Private**. In questo caso ho scelto di mantenere attivo l'accesso per gli utenti guest, ma puoi disabilitarlo in base alle policy della tua organizzazione.

Salva l'etichetta.

![Creazione label](/images/posts/governance/create-label.png)

---

### 3. Creare e pubblicare una Label Policy

L'etichetta appena creata non è ancora visibile agli utenti: per renderla operativa è necessario pubblicarla tramite una **Label Policy**.

Vai alla sezione **Label Policies** nel Purview Portal e crea una nuova policy includendo:

- La label `Collaborazione interna – Gruppi privati`.
- Come **target**, puoi scegliere **tutti gli utenti** oppure una **Distribution List** specifica.
- Nelle **Default settings for Groups & Sites**, imposta la label come predefinita e spunta l'opzione **Require users to apply a label to their groups or sites** — questo passaggio è fondamentale per ottenere l'effetto desiderato.

Assegna un nome e una descrizione alla policy, quindi pubblicala.

---

### 4. Come escludere singoli utenti o gruppi dalla policy

Se vuoi applicare la policy all'intera organizzazione ma mantenere alcuni utenti — ad esempio un gruppo IT che gestisce richieste tramite ticket o portali interni — liberi di creare gruppi pubblici, puoi farlo esclusivamente tramite PowerShell.

L'interfaccia grafica del Purview Compliance Portal **non supporta le eccezioni** quando la policy è configurata con il target *All*: la funzionalità esiste solo lato PowerShell.

**Prerequisiti — connessione al modulo Security & Compliance:**

```powershell
Connect-IPPSSession
```

**Escludere singoli utenti:**

```powershell
Set-LabelPolicy -Identity "Nome della tua Policy" `
    -AddExchangeLocationException "utente1@dominio.com", "utente2@dominio.com"
```

**Escludere tutti i membri di una Distribution List o di un gruppo:**

```powershell
[array]$Members = Get-DistributionGroupMember -Identity "Nome del gruppo IT" |
    Where-Object { $_.RecipientTypeDetails -eq "UserMailbox" } |
    Select-Object -ExpandProperty PrimarySmtpAddress

Set-LabelPolicy -Identity "Nome della tua Policy" -AddExchangeLocationException $Members
```

**Verificare le eccezioni attive sulla policy:**

```powershell
[array]$Exclusions = Get-LabelPolicy -Identity "Nome della tua Policy" |
    Select-Object -ExpandProperty ExchangeLocationException
$Exclusions.Name
```

**Rimuovere un'eccezione:**

```powershell
Set-LabelPolicy -Identity "Nome della tua Policy" `
    -RemoveExchangeLocationException "utente1@dominio.com"
```

> **Nota**: Le eccezioni non hanno effetto immediato. I client Outlook e Teams aggiornano la cache dall'Information Protection Service con i propri tempi; potrebbe essere necessario attendere prima che la modifica sia visibile agli utenti esclusi.

---

### 5. Propagazione e verifica

Le modifiche non vengono applicate istantaneamente: la policy può impiegare **fino a 24 ore** per propagarsi a tutti gli utenti e servizi del tenant.

Una volta completata la propagazione, il comportamento sarà il seguente:

- Durante la creazione di un Team, gli utenti vedranno la label predefinita `Collaborazione interna – Gruppi privati` già selezionata.
- L'opzione per creare un Team **pubblico sarà disabilitata** (grigia e non selezionabile).
- Non sarà possibile rimuovere la label o selezionare "None", se la policy è configurata correttamente con l'etichetta obbligatoria.

**Creazione di un Team in Microsoft Teams:**

![Label applicata in Teams](/images/posts/governance/block-public-teams.png)

**Creazione di un Team Site in Microsoft SharePoint Online:**

![Label applicata in SharePoint](/images/posts/governance/block-public-site.png)

---

## Punti critici e suggerimenti

**Etichetta obbligatoria e priorità della policy** — per evitare che gli utenti aggirano la configurazione selezionando "None", assicurati che l'etichetta sia marcata come obbligatoria nella policy e che questa abbia la priorità più alta rispetto ad eventuali altre policy attive nel tenant.

**Licensing** — alcune funzionalità avanzate delle Sensitivity Labels (come la crittografia dei contenuti o specifiche integrazioni con la compliance) potrebbero richiedere licenze più avanzate, come Microsoft 365 E5, Teams Premium o add-on di conformità dedicati.

**Test prima del rollout** — prima di applicare la configurazione all'intera organizzazione, testala sempre su un ambiente pilota o su un gruppo ristretto di utenti. Le 24 ore di propagazione rendono i cicli di correzione lenti, quindi è meglio validare la configurazione in anticipo.

---

## Conclusione

Le Sensitivity Labels sono uno strumento potente per governare come vengono creati gli spazi collaborativi in Microsoft 365. Con la configurazione descritta in questo articolo, è possibile garantire che tutti i nuovi Teams e siti SharePoint nascano con un livello di privacy controllato, riducendo il rischio di esposizione accidentale dei dati e semplificando la gestione della compliance.

Il tutto senza sacrificare la flessibilità: grazie alle eccezioni via PowerShell, è sempre possibile riservare maggiori privilegi a chi ne ha realmente bisogno.

---
