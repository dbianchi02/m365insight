---
title: "Blocco della creazione di gruppi pubblici in Microsoft Teams tramite Sensitivity Labels"
date: 2025-11-20T10:00:00+02:00
draft: false
tags: ["Microsoft Teams", "Sensitivity Labels", "Governance", "Sicurezza M365", "Microsoft Purview"]
description: "Scopri come impedire la creazione di team pubblici in Microsoft Teams utilizzando le Sensitivity Labels di Microsoft Purview, garantendo maggiore controllo e sicurezza."
cover:
    image: "/images/posts/block-public-teams.png" # Immagine in evidenza per l'articolo
    caption: "Gestisci la privacy dei tuoi team con le Sensitivity Labels"
---

## Introduzione

In un ambiente di lavoro moderno e collaborativo come Microsoft Teams, la facilità di creazione di nuovi spazi può portare a sfide di governance e sicurezza. Uno dei rischi maggiori è la proliferazione incontrollata di team pubblici, che potrebbero esporre dati sensibili o creare confusione organizzativa. Questo articolo ti guiderà attraverso una soluzione efficace: l'utilizzo delle **Sensitivity Labels (etichette di sensibilità)** di Microsoft Purview Information Protection per imporre la privacy dei gruppi fin dalla loro creazione.

---

### Perché utilizzare le Sensitivity Labels? 🏷️

In Microsoft Teams, la creazione di un team è strettamente legata alla creazione di un gruppo Microsoft 365 (O365 Group). Se si desidera impedire agli utenti di creare team pubblici, ma consentire la creazione di team privati, la soluzione non è limitare la creazione di gruppi *tout court*, bensì imporre una configurazione di privacy tramite etichette di sensibilità (Sensitivity Labels) gestite con **Microsoft Purview Information Protection**.

Questo approccio offre un controllo granulare, permettendo agli amministratori di definire le regole di utilizzo e la privacy per i dati e i contenitori, senza bloccare la produttività degli utenti.

---

### Prerequisiti ✅

Prima di iniziare, assicurati di avere i seguenti requisiti:

* **Licenza Microsoft 365** che supporti Sensitivity Labels per contenitori (Teams, SharePoint, Groups). Tipicamente, questo include licenze come Microsoft 365 E3 (con add-on) o E5, o le versioni equivalenti.
* Accesso al **Microsoft Purview Compliance Portal** (attualmente `https://compliance.microsoft.com`).
* Permessi di amministratore per creare e pubblicare etichette.

---

### Procedura Passo-Passo ⚙️

Segui questi passaggi dettagliati per configurare e pubblicare le tue etichette di sensibilità.

#### 1. Abilitare le Sensitivity Labels per Teams e Gruppi

Per prima cosa, verifica che la funzionalità sia attiva nel tuo tenant.

* Accedere al **Microsoft Purview Compliance Portal** → **Information Protection** → **Labels**.
* Verificare che la funzionalità sia attiva per **Teams, Groups e Sites**.
    * *Nota:* Se non lo è, potresti dover seguire la [guida ufficiale Microsoft](https://learn.microsoft.com/en-us/microsoft-365/compliance/enable-sensitivity-labels-for-containers?view=o365-worldwide) per abilitarla via PowerShell.

#### 2. Creare una nuova Sensitivity Label

Ora creeremo l'etichetta che imporrà la privacy.

* Cliccare su **Create a label**.
* Impostare lo **Scope** su: **Groups & Sites** (questo include automaticamente Teams e O365 Groups).
* Configurare le impostazioni della label:
    * **Privacy**: Selezionare **Private**.
    * **External access**: Disabilitare se necessario (es. blocco utenti guest).
* Salvare la label con un nome chiaro e descrittivo (es. `Private Only` o `Team Interno Riservato`).

#### 3. Creare e pubblicare una Label Policy

Una volta creata l'etichetta, è necessario pubblicarla tramite una policy per renderla disponibile agli utenti.

* Tornare alla sezione **Label Policies** nel Compliance Portal.
* Creare una nuova policy e includere:
    * La label appena creata (es. `Private Only`).
    * Impostare **Mandatory labeling for Groups & Sites** su **On** (questo è obbligatorio per l'effetto desiderato).
    * **Default label**: Impostare la label `Private Only` come predefinita.
* Selezionare il **target**:
    * **Tutti gli utenti** o un **Distribution List (DL)** contenente gli utenti a cui vuoi applicare questa limitazione.
* Pubblicare la policy.

#### 4. Propagazione e Verifica 🕒

Le modifiche non sono immediate.

* La policy può impiegare **fino a 24 ore** per essere applicata a tutti gli utenti e per propagarsi attraverso i servizi.
* **Dopo la propagazione:**
    * Gli utenti vedranno la label predefinita (`Private Only`) durante la creazione di un Team.
    * L’opzione per creare un Team **pubblico sarà disabilitata (grigia)**.
    * **Non sarà possibile modificare la label o selezionare “None”** (se la policy è configurata correttamente con l'etichetta mandatoria).

---

### Punti Critici e Suggerimenti 💡

* **Rimuovere l’opzione “None”**: Per assicurarti che gli utenti non possano bypassare la policy, verifica che l'etichetta sia impostata come obbligatoria nella policy e che quest'ultima abbia la priorità più alta, se ci sono altre policy attive.
* **Licensing**: Alcune funzionalità avanzate delle Sensitivity Labels (come la protezione dei contenuti crittografati o specifiche integrazioni) potrebbero richiedere licenze più avanzate (es. Microsoft 365 E5, Teams Premium, o add-on di conformità).
* **Verifica**: Testa sempre la configurazione in un ambiente pilota o con un piccolo gruppo di utenti prima di applicarla a tutta l’organizzazione.

---

### Benefici 🚀

L'implementazione delle Sensitivity Labels per la creazione di team pubblici offre numerosi vantaggi:

* **Governance semplificata**: Garantisce che tutti i nuovi Teams siano creati con un livello di privacy predefinito e controllato.
* **Sicurezza rafforzata**: Riduce il rischio di esposizione accidentale o intenzionale di dati sensibili in gruppi pubblici.
* **Compliance**: Facilita l'integrazione delle policy di protezione dati aziendali, aiutando a mantenere la conformità normativa.

---

## Conclusione

Le Sensitivity Labels rappresentano uno strumento potente e flessibile per la governance e la sicurezza dei dati in Microsoft 365. Implementando questa procedura, potrai esercitare un controllo significativo sulla creazione dei team in Microsoft Teams, assicurando che i tuoi spazi collaborativi rispettino gli standard di privacy e protezione della tua organizzazione.

---