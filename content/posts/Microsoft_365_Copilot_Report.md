---
title: "Monitoraggio di Microsoft 365 Copilot: Report dettagliati e dashboard aggregate"
date: 2025-11-28T13:00:00+01:00
draft: false
tags: ["adoption-m365", "Microsoft 365 Copilot", "Monitoraggio"]
description: "Scopri come monitorare Microsoft 365 Copilot con report nominali e dashboard aggregate per ottimizzare adozione e governance."
authors: ["Davide Bianchi"]
---

## Introduzione

L’adozione di **Microsoft 365 Copilot** è una delle innovazioni più significative degli ultimi anni, ma senza un monitoraggio efficace rischia di diventare una “scatola nera”. Le aziende devono sapere **chi utilizza Copilot**, **in quali app** e **con quale frequenza**, per garantire **governance**, **sicurezza** e **valore di business**.

Microsoft mette a disposizione due approcci principali:

* **Report dettagliati e nominali**: tramite il **Microsoft 365 Copilot Usage Report**, che mostra quali utenti hanno utilizzato Copilot, in quali app e con quale volume.
* **Dati aggregati**: tramite la **Copilot Dashboard** e le dashboard di **Viva Insights Analyst**, che offrono una vista sull’adozione complessiva e i trend.

---

## Perché monitorare Copilot?

Monitorare Copilot non è solo una questione tecnica, ma strategica:

* **Governance e compliance**: verificare che l’uso sia conforme alle policy interne e al GDPR.
* **Misurazione del ROI**: capire se l’investimento in Copilot genera produttività reale.
* **Ottimizzazione licenze**: individuare utenti inattivi e riallocare risorse.
* **Supporto al Change Management**: identificare team che necessitano formazione o supporto.

---

## Prerequisiti

Prima di iniziare, assicurati di avere:

* Licenze **Microsoft 365 Copilot** attive nel tenant.
* Ruolo **Global Admin** o **Reports Reader** per accedere ai report.
* Abilitazione della **Copilot Dashboard** in Microsoft 365 Admin Center (Viva Insights Admin Settings).
* Accesso a **Viva Insights Analysis** per le dashboard di adozione (Necessario ruolo Insights Administrator o Insisghts Analyst).

---

## Panoramica della soluzione

Il monitoraggio si basa su due strumenti complementari:

1. **Copilot Usage Report**  
   Fornisce dati nominali per singolo utente e app. Ideale per analisi puntuali e verifiche di compliance.
   
2. **Copilot Dashboard e Viva Insights**  
   Offrono dati aggregati e trend di adozione. Perfette per presentazioni al management e per definire KPI.

---

## Procedura passo-passo

### 1. Accedere al Copilot Usage Report

1. Vai su **Microsoft 365 Admin Center** → **Reports > Usage > Microsoft 365 Copilot**.
2. È possibile filtrare per:
   * **Utente** (nome, UPN)
   * **App** (Word, Excel, Teams, Outlook, ecc.)
   * **Periodo** (giorni, settimane)
3. Esporta il report in **CSV** per analisi avanzate in Excel o Power BI.

**Nota:** Questo report è **nominale**, quindi soggetto alle policy di privacy aziendali. Assicurati di informare gli utenti e rispettare le normative GDPR.

![CopilotUsageReport](/images/posts/analytics/microsoft-365-copilot-usage.png)

---

### 2. Consultare la Copilot Dashboard

La **Copilot Dashboard** è disponibile in Viva Insights Personal (se abilitata) e mostra:

* Numero totale di prompt inviati.
* Distribuzione per app.
* Trend di utilizzo nel tempo.

**Attenzione:** Non mostra dati nominali, solo aggregati. È utile per capire l’adozione complessiva.

![CopilotDashboard](/images/posts/analytics/CopilotDashboard.png)

---

### 3. Analizzare le dashboard di Viva Insights Analyst

Se hai i permessi di Insight Analyst o di Insight Administrator, puoi accedere al portale **Viva Insights Analysis** ed a report dedicati come:

* **Copilot Adoption**: percentuale di utenti attivi.
* **Copilot Engagement**: frequenza di utilizzo e correlazioni con produttività.

Oltre alla consultazione manuale, la reportistica di Viva Insights Analyst offre opzioni avanzate che la rendono ideale per analisi continue e personalizzate:

Schedulazione automatica: i report possono essere pianificati per l’esecuzione giornaliera, settimanale o mensile, riducendo il lavoro manuale.
Esportazione e integrazione con Power BI: i dati possono essere esportati e ripubblicati in Power BI per creare dashboard interattive e analisi avanzate.
Filtri e segmentazione: è possibile filtrare i report per ruolo, reparto, area geografica o gruppo di utenti.
Creazione di report personalizzati: Insights Analyst consente di combinare metriche e KPI per costruire report su misura, allineati agli obiettivi di business.

**Suggerimento**: sfrutta queste funzionalità per automatizzare il monitoraggio e presentare al management dashboard sempre aggiornate, senza interventi manuali.

---

## Punti critici e buone pratiche

* **Abilitazione dashboard**: non è automatica, richiede adeguata configurazione in Microsoft 365 Admin Center.
* **Interpretazione dei dati**: non fermarti ai numeri; correlali con obiettivi di business e feedback qualitativi.
* **Formazione**: se l’adozione è bassa, pianifica training mirati e campagne di awareness.

---

## Benefici

* **Visibilità completa**: dal singolo utente alla panoramica aziendale.
* **Decisioni informate**: supporto per strategie di adozione e governance.
* **Ottimizzazione costi**: identificare licenze inutilizzate.
* **Governance proattiva**: prevenire rischi di compliance e uso improprio.

---

### Link utili

* [Copilot Usage Report – Microsoft Docs](https://learn.microsoft.com/en-us/microsoft-365/admin/activity-reports/microsoft-365-copilot-usage?view=o365-worldwide)
* [Copilot Dashboard – Microsoft Docs](https://learn.microsoft.com/en-us/viva/insights/org-team-insights/copilot-dashboard)
* [Viva Insights Analyst – Microsoft Docs](https://learn.microsoft.com/it-it/viva/insights/introduction)

---

## Conclusione

Il monitoraggio di Microsoft 365 Copilot è un pilastro della governance moderna. Combinando **report dettagliati** e **dashboard aggregate**, le aziende possono misurare l’adozione, garantire conformità e massimizzare il valore dell’investimento.
