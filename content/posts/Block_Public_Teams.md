---
title: "Blocco della creazione di Gruppi M365 Pubblici in Microsoft Teams e SharePoint Online tramite Sensitivity Labels"
date: 2026-02-14T10:00:00+02:00
draft: false
tags: ["Governance", "Microsoft Teams", "Sensitivity Labels", "Sicurezza M365", "Microsoft Purview"]
authors: ["Davide Bianchi"]
description: "Scopri come impedire la creazione di Gruppi Microsoft 365 Pubblici in Microsoft Teams utilizzando le Sensitivity Labels di Microsoft Purview, garantendo maggiore controllo e sicurezza."
---

## Introduzione

In un ambiente di lavoro moderno e collaborativo come Microsoft Teams, la facilità di creazione di nuovi spazi può portare a sfide di governance e sicurezza. Uno dei rischi maggiori è la proliferazione incontrollata di team pubblici, che potrebbero esporre dati sensibili o creare confusione organizzativa. Questo articolo ti guiderà attraverso una soluzione efficace: l'utilizzo delle **Sensitivity Labels (etichette di sensibilità)** di Microsoft Purview Information Protection per imporre la privacy dei gruppi fin dalla loro creazione.

---

### Perché utilizzare le Sensitivity Labels?

In Microsoft Teams, la creazione di un team è strettamente legata alla creazione di un gruppo Microsoft 365. Se si desidera impedire agli utenti di creare gruppi pubblici all'intera organizzazione, ma consentire la creazione di team privati, la soluzione non è limitare la creazione di gruppi *tout court*, bensì imporre una configurazione di privacy tramite Sensitivity Labels gestite con **Microsoft Purview Information Protection**.

Questo approccio offre un controllo granulare, permettendo agli amministratori di definire le regole di utilizzo e la privacy per i dati, senza bloccare produttività e collaborazione degli utenti.

---

### Prerequisiti

Prima di iniziare, assicurati di avere i seguenti requisiti:

* **Licenza Microsoft 365** che supporti Sensitivity Labels (Teams, SharePoint, Groups). Tipicamente, questo include licenze come Microsoft 365 E3 (con add-on) o E5, o le versioni equivalenti.
* Accesso al **Microsoft Purview Compliance Portal** (attualmente `https://compliance.microsoft.com`).
* Permessi di amministratore per creare e pubblicare etichette.

---

### Procedura Passo-Passo

Segui questi passaggi dettagliati per configurare e pubblicare le tue etichette di sensibilità.

#### 1. Abilitare le Sensitivity Labels per Teams e Gruppi

Per prima cosa, verifica che la funzionalità sia attiva nel tuo tenant.

* Accedere al **Microsoft Purview Compliance Portal** → **Information Protection** → **Labels**.
![Purview Portal](/images/posts/governance/label-portal.png)
* Verificare che la funzionalità sia attiva per **Teams, Groups e Sites**.
    * *Nota:* Se non lo è, potresti dover seguire la [guida ufficiale Microsoft](https://learn.microsoft.com/en-us/entra/identity/users/groups-assign-sensitivity-labels?tabs=microsoft#enable-sensitivity-label-support-in-powershell) per abilitarla via PowerShell.

#### 2. Creare una nuova Sensitivity Label

Ora creeremo l'etichetta che imporrà la privacy.

* Cliccare su **Create a label**.
* Definire i parametri di configurazione in modo che siano parlanti sia per gli admin che accedono al portale di Purview, sia per gli utenti che visualizzeranno la label (in questo caso ho scelto di configurarla in questo modo:
    * **Name**: Block Public Teams
    * **Display Name** (nome visualizzato dagli utenti): Collaborazione interna – Gruppi privati
    * **Description for users**: Usa questa etichetta per creare Team, Gruppi e Siti SharePoint visibili solo ai membri. L'organizzazione non consente la creazione di gruppi pubblici.
    * **Description for admins**: Questa etichetta imposta la privacy dei Gruppi Microsoft 365, Team e siti SharePoint su **Privato**, impedendo la creazione di gruppi pubblici.
* Impostare lo **Scope** su: **Groups & Sites** (questo include automaticamente Teams e O365 Groups).
* Configurare le impostazioni della label:
    * **Protection Settings**: Selezionare **Privacy and external user access**.
    * **Privacy**: Selezionare **Private**. Qui è possibile anche scegliere se permettere l'accesso di utenti Guest, ma in questo caso ho scelto di mantenere l'accesso di utenti esterni atttivo.
* infine salvare la label.

![creation](/images/posts/governance/create-label.png)

#### 3. Creare e pubblicare una Label Policy

Una volta creata l'etichetta, è necessario pubblicarla tramite Label Policy per renderla disponibile agli utenti.

* Tornare alla sezione **Label Policies** nel Purview Portal.
* Creare una nuova policy e includere:
    * La label appena creata (es. `Collaborazione interna – Gruppi privati`).
    * Selezionare il **target**:
    * **Tutti gli utenti** o un **Distribution List (DL)** contenente gli utenti a cui vuoi applicare questa limitazione.
    * Impostare all'interno delle **Default settings for Groups & Sites**:
    * **Default label**: Impostare la label `Collaborazione interna – Gruppi privati` come predefinita. 
    * **Require users to apply a label to theri groups or sites**: Selezionare la spunta (questo è obbligatorio per l'effetto desiderato).
    * Infine assegnare un nome e una description parlante alla policy, e procedere con la pubblicazione
* Pubblicare la policy.

#### 4. Come escludere singoli utenti/gruppo di utenti dalla policy appena creata?

Qualora voleste applicare la policy all'intera organizzazione, ma mantenere un gruppo di persone abilitato (es. un gruppo IT che su richiesta tramite ticket/portali interni) alla creazione di gruppi pubblici per questioni organizzative, è possibile farlo esclusivamente tramite PowerShell, poiché l'interfaccia grafica del Purview Compliance Portal **non supporta le eccezioni** quando la policy è configurata con il target *All*.

**Prerequisiti**: Connessione al modulo PowerShell di Security & Compliance:
```powershell
Connect-IPPSSession
```

**Escludere singoli utenti:**
```powershell
Set-LabelPolicy -Identity "Nome della tua Policy" -AddExchangeLocationException "utente1@dominio.com", "utente2@dominio.com"
```

**Escludere tutti i membri di una Distribution List o di un gruppo:**
```powershell
[array]$Members = Get-DistributionGroupMember -Identity "Nome del gruppo IT" | Where-Object {$_.RecipientTypeDetails -eq "UserMailbox"} | Select-Object -ExpandProperty PrimarySmtpAddress

Set-LabelPolicy -Identity "Nome della tua Policy" -AddExchangeLocationException $Members
```

**Verificare le eccezioni attive sulla policy:**
```powershell
[array]$Exclusions = Get-LabelPolicy -Identity "Nome della tua Policy" | Select-Object -ExpandProperty ExchangeLocationException
$Exclusions.Name
```

**Rimuovere un'eccezione:**
```powershell
Set-LabelPolicy -Identity "Nome della tua Policy" -RemoveExchangeLocationException "utente1@dominio.com"
```

> **Nota**: Le eccezioni non hanno effetto immediato. I client Outlook e Teams aggiornano la cache dall'Information Protection Service con i propri tempi, pertanto potrebbe essere necessario attendere prima che la modifica sia visibile agli utenti esclusi.

#### 5. Propagazione e Verifica

Le modifiche non sono immediate.

* La policy può impiegare **fino a 24 ore** per essere applicata a tutti gli utenti e per propagarsi attraverso i servizi.
* **Dopo la propagazione:**
    * Gli utenti vedranno la label predefinita (`Collaborazione interna – Gruppi privati`) durante la creazione di un Team.
    * L’opzione per creare un Team **pubblico sarà disabilitata (grigia)**.
    * **Non sarà possibile modificare la label o selezionare “None”** (se la policy è configurata correttamente con l'etichetta obbligatoria).

Use case creazione di un Team in Microsoft Teams:

![Label Appicata Teams](/images/posts/governance/block-public-teams.png)

Use case creazione di un Team Site in Microsoft SharePoint Online:

![Label Appicata SharePoint](/images/posts/governance/block-public-site.png)

---

### Punti Critici e Suggerimenti

* **Rimuovere l’opzione “None” ed impostare una default label**: Per assicurarti che gli utenti non possano bypassare la policy, verifica che l'etichetta sia impostata come obbligatoria nella policy e che quest'ultima abbia la priorità più alta, se ci sono altre policy attive.
* **Licensing**: Alcune funzionalità avanzate delle Sensitivity Labels (come la protezione dei contenuti crittografati o specifiche integrazioni) potrebbero richiedere licenze più avanzate (es. Microsoft 365 E5, Teams Premium, o add-on di conformità).
* **Verifica**: Testa sempre la configurazione in un ambiente pilota o con un piccolo gruppo di utenti prima di applicarla a tutta l’organizzazione.

---

### Benefici

L'implementazione delle Sensitivity Labels per la creazione di team pubblici offre numerosi vantaggi:

* **Governance semplificata**: Garantisce che tutti i nuovi Teams siano creati con un livello di privacy predefinito e controllato.
* **Sicurezza rafforzata**: Riduce il rischio di esposizione accidentale o intenzionale di dati sensibili in gruppi pubblici.
* **Compliance**: Facilita l'integrazione delle policy di protezione dati aziendali, aiutando a mantenere la conformità normativa.

---

## Conclusione

Le Sensitivity Labels rappresentano uno strumento potente e flessibile per la governance e la sicurezza dei dati in Microsoft 365. Implementando questa procedura, potrai esercitare un controllo significativo sulla creazione dei team in Microsoft Teams, assicurando che i tuoi spazi collaborativi rispettino gli standard di privacy e protezione della tua organizzazione.

---