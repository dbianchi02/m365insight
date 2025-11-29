---
title: "Come analizzare i log di Microsoft Entra ID per individuare compromissioni account 🛡️ "
date: 2025-11-24T15:00:00+01:00
draft: false
tags: ["identità-sicurezza", "Microsoft 365 Copilot", "Copilot Studio", "Entra ID"]
authors: ["Davide Bianchi"]
description: "Scopri come configurare l'agente in Microsoft 365 Copilot per automatizzare l'analisi dei log di Entra ID, individuare accessi anomali e generare risposte strutturate per l'Incident Response."
---

## Introduzione

La **sicurezza degli account aziendali** è una priorità assoluta. Gli accessi anomali, provenienti da location insolite o dispositivi non autorizzati, possono essere indicatori cruciali di una compromissione. In questo articolo, vedremo come configurare un'agente Copilot personalizzato **M365 – Compromissioni IT Sec** per analizzare i log di Microsoft Entra ID (precedentemente Azure AD) e generare risposte tecniche strutturate, pronte per i team di sicurezza.

![CopilotAgent](/images/posts/security/agente-copilot.png)

---

## Perché Serve un Agente Dedicato?

Gli strumenti nativi di Microsoft Entra ID forniscono informazioni dettagliate sui sign-in, ma spesso manca un'analisi automatizzata e contestuale che si concentra sull'identificazione rapida degli incidenti.

Un agente dedicato colma questa lacuna offrendo:

* **Evidenza di Pattern Abituali:** Identifica e stabilizza i pattern di accesso tipici per un utente (location, device, browser).
* **Identificazione di Eventi Sospetti:**
    * Accessi da paesi esteri o IP associati a VPN/proxy.
    * Browser o sistemi operativi insoliti.
    * Tentativi MFA anomali o sospetti.
* **Generazione di Timeline:** Ricostruisce una timeline dettagliata degli eventi critici per una facile indagine.
* **Risposta Strutturata (Incident Response):** Produce una risposta tecnica con conclusione e azioni consigliate.

Questo agente Copilot nasce per automatizzare l'analisi e **ridurre il tempo di detection e risposta (MTTR)**.

---

## Come Configurare l'Agente in Microsoft 365 Copilot

La chiave per l'efficacia dell'agente risiede nella sua configurazione in Copilot Studio (o Microsoft 365 Copilot, a seconda dell'integrazione).

### 1. Fonti Necessarie

L'agente lavora analizzando i dati grezzi dei log. Prepara i seguenti elementi:

* **File CSV dei Sign-in Logs:** Esportato direttamente da Microsoft Entra ID.
    * Vai su **Microsoft Entra ID** → **Monitoring & health** → **Sign-in logs**.
    * Seleziona il periodo di interesse (es. ultimi 7 giorni) e clicca su **Download CSV**.
* **Documentazione Ufficiale Microsoft (per contesto):**
    * [Sign-in Logs Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)
    * [Conditional Access Policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)

### 2. Creazione dell'Agente in Copilot Studio

Durante la creazione dell'agente personalizzato, definisci con precisione le sue istruzioni.

| Campo | Valore |
| :--- | :--- |
| **Nome Agente** | M365 – Compromissioni IT Sec |
| **Descrizione** | Analizza i log di accesso Microsoft Entra ID per individuare anomalie e possibili compromissioni, fornendo una risposta tecnica pronta per effettuare una remediation. |

#### Istruzioni Principali (Prompt di Sistema)

* Analizza i file CSV dei sign-in logs Microsoft Entra ID.
* Identifica i pattern di accesso abituali (location, device, browser).
* Evidenzia gli accessi anomali (location insolite, IP VPN/proxy, browser non usuali).
* Ricostruisci la timeline degli eventi sospetti.
* Fornisci una sintesi tecnica con:
    * Sintesi accessi abituali.
    * Evidenza anomalie.
    * Timeline eventi sospetti.
    * **Conclusione tecnica** (es. "Rischio di compromissione alto/basso").
    * **Azioni consigliate** (remediation).
* Utilizza **tabelle** e **bullet point** per massima chiarezza e leggibilità.
* Se mancano dati chiave per l'analisi (es. il campo IP è vuoto), segnala la limitazione all'utente.

![CopilotStudio](/images/posts/security/build-agent.png)
---

## Prompt Suggeriti per Risposte Efficaci

Una volta che l'agente è pronto, l'efficacia della risposta dipende dalla chiarezza del prompt utente.

| Prompt | Focus |
| :--- | :--- |
| **Prompt 1 – Analisi completa con timeline e ticket** | Analisi di anomalie comportamentali, IP sospetti, timeline completa e output formattato per il ticket. |
| **Prompt 2 – Focus su IP e MFA** | Analisi mirata su indicatori tecnici di attacco (VPN, proxy, tentativi MFA). |
| **Prompt 3 – Report sintetico per manager** | Sintesi di alto livello per una panoramica rapida, con enfasi sul rischio e le raccomandazioni. |

### Il Prompt Perfetto per una Risposta Completa

Questo prompt combina tutti gli elementi necessari per un'analisi forense preliminare:

> "Analizza il file di sign-in logs allegato.
> Voglio sapere se ci sono accessi riusciti da location, device o browser insoliti rispetto al comportamento abituale dell’utente.
> Evidenzia eventuali accessi da IP associati a VPN, proxy o paesi esteri.
> Fornisci una timeline dettagliata degli eventi sospetti e una risposta pronta da inserire in un ticket ServiceNow, comprensiva di conclusione tecnica e azioni consigliate."

---

## Best Practice per Risposte Corrette

Per assicurare che l'agente fornisca i risultati più accurati e utili:

* **Completa i Log:** Più campi Entra ID sono inclusi nel CSV, più l'analisi comportamentale sarà accurata.
* **Confronta con Baseline:** L'agente è efficace perché identifica le anomalie rispetto al **comportamento abituale** dell'utente, non solo rispetto a regole statiche.
* **Richiedi Azioni Consigliate:** Trasforma l'analisi (**il cosa è successo**) in una remediation concreta (**il cosa fare**).
* **Integra con Policy di Sicurezza:** Le analisi dovrebbero sempre essere contestualizzate rispetto a policy aziendali attive (es. Conditional Access, MFA enforcement, geofencing).

## Conclusione

L'agente Copilot configurato è uno strumento potente per accelerare la risposta agli incidenti di sicurezza. Automatizzando l'analisi dei log grezzi di Entra ID e fornendo insight immediati, riduce il tempo di detection e migliora significativamente la protezione degli account. Configurarlo correttamente in Microsoft 365 Copilot è il passo decisivo per trasformare una montagna di dati grezzi in **decisioni operative immediate**.