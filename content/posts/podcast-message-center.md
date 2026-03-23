---
title: "Da Audio Overview a Podcast Automatico: come ho trasformato il Message Center di Microsoft 365 in un briefing mattutino su Telegram"
date: 2026-03-23T07:00:00+02:00
draft: false
tags: ["Azure AI Foundry", "Microsoft 365", "Automazione", "Message Center", "PowerShell"]
description: "Come ho costruito un sistema completamente automatizzato che ogni mattina genera un podcast a due voci dal Message Center di Microsoft 365 e lo invia su Telegram, usando PowerShell e Azure AI Foundry."
---

## Il problema: troppi messaggi, troppo poco tempo

Chi gestisce un tenant Microsoft 365 lo sa bene: il Message Center è uno strumento indispensabile, ma tenersi aggiornati quotidianamente su ogni annuncio, aggiornamento pianificato e ritiro di funzionalità è un'attività che richiede tempo e concentrazione. Non si tratta solo di leggere — bisogna capire cosa cambia, chi è impattato, se è richiesta un'azione e con quale scadenza.

---

## L'idea: un podcast mattutino

Tutto è partito da una funzione che avevo scoperto in Microsoft 365 Copilot: **Audio Overview**. La possibilità di caricare un documento su SharePoint o OneDrive e ottenere una sintesi audio — con tanto di modalità podcast a due voci — mi aveva colpito immediatamente. Sarebbe stato perfetto: ascoltare il riepilogo degli aggiornamenti in macchina, durante il tragitto verso l'ufficio.

Il problema è che **Audio Overview non è disponibile come action in Power Automate**. Non esiste ad oggi un connettore che permetta di automatizzarne la generazione. L'unica modalità è manuale: carichi il documento contenente tutti i nuovi messaggi, clicchi genera, ascolti. Utile, ma non automatizzabile.

Da lì ho iniziato a ragionare su come replicare quella stessa esperienza in modo completamente autonomo, senza intervento umano.

---

## La soluzione: quattro script, un podcast

L'architettura che ho costruito si articola in quattro passaggi eseguiti ogni mattina in sequenza, orchestrati da uno script master.

![PodcastWorkflow](/images/posts/automazione/podcast-flow-diagram.png)

### 1. Raccolta dei messaggi

Il punto di partenza è una chiamata alle **Microsoft Graph API** con il permesso `ServiceMessage.Read.All`, autenticata tramite un'App Registration con certificato. I messaggi delle ultime 24 ore vengono estratti e salvati in un file JSON strutturato con titolo, servizio coinvolto, tag, date e corpo del testo.

Il JSON diventa il dato grezzo su cui lavorano tutti i passaggi successivi.

### 2. Generazione dello script podcast

Il file JSON viene passato a **GPT-4o mini su Azure AI Foundry**. Ho scelto GPT-4o mini per ragioni pratiche: il task è di sintesi e riscrittura, non richiede ragionamento complesso, e il costo è nell'ordine dei pochi centesimi al mese sul volume giornaliero che genero.

Il prompt che ho costruito nel tempo definisce due voci distinte:

- **Alex**: il conduttore, tono da radio mattutina, diretto e con una voce limpida.
- **Sara**: l'esperta tecnica, risponde con precisione, cita date, spiega impatti pratici e azioni richieste.

Una delle sfide più interessanti è stata insegnare al modello a **prioritizzare i messaggi correttamente**: i retirement con scadenze immediate devono venire per primi e ricevere più spazio, le nuove funzionalità senza impatto immediato possono essere liquidate in due battute. Ho risolto pre-ordinando i messaggi nel codice PowerShell — divisi in tre gruppi (Retirement, Admin impact, Informativi) — prima ancora di passarli al modello, in modo che l'ordine nel JSON già rifletta la priorità.

Il risultato è un file `.txt` con lo script del podcast, pronto per la sintesi vocale.

### 3. Conversione in audio

Lo script testuale viene passato riga per riga al modello **gpt-4o-mini-tts**, sempre su Azure AI Foundry. Per ogni battuta riconoscibile come `ALEX:` o `SARA:`, viene fatta una chiamata separata all'API TTS con la voce corrispondente (`echo` per Alex, `nova` per Sara) e istruzioni stilistiche specifiche per ciascuna. I chunk audio vengono poi concatenati in un unico file `.mp3`.

La scelta di usare **gpt-4o-mini-tts invece di Azure Speech Service** — che avevo inizialmente considerato, essendo già nella subscription — è stata dettata dalla qualità. Il modello TTS di OpenAI, pur essendo accessibile tramite la stessa infrastruttura Azure, offre una naturalezza notevolmente superiore, in particolare sul ritmo e sulla gestione delle pause.

### 4. Consegna su Telegram

L'ultima parte della pipeline invia l'mp3 direttamente a un **gruppo Telegram** tramite un bot dedicato. La scelta di Telegram rispetto ad un workflow Teams, ad un salvataggio in una SharePoint DL o tramite mail è stata deliberata: Teams, Outlook e SharePoint/OneDrive richiederebbero l'apertura di app di lavoro. Telegram è accessibile in un solo tap, funziona benissimo come lettore audio su smartphone, e il bot creato con BotFather può essere aggiunto ad gruppo ed essere condiviso con i colleghi in modo immediato.

![TelegramBot](/images/posts/automazione/telegram-bot.png)

---

## Come ho schedulato tutto

L'intera pipeline gira su una **VM Windows Server 2025 in Azure** che avevo già a disposizione, gestita tramite due componenti:

**Azure Automation** si occupa del ciclo di vita della VM stessa: un runbook PowerShell avvia la macchina alle 06:50 dal lunedì al venerdì e la spegne alle 08:10, una volta completata la pipeline. Questo significa che la VM rimane accesa meno di un'ora e mezza al giorno, contenendo i costi.

**Task Scheduler** sulla VM esegue lo script master alle 07:00. Il master chiama i quattro script in sequenza, controlla i codici di uscita, registra tutto in un log giornaliero e — se qualcosa va storto — può inviare una notifica di errore.

Il risultato è che ogni mattina dal lunedì al venerdì, senza alcun intervento, il podcast è disponibile nel gruppo Telegram entro le 07:30 circa.

---

## Componenti tecnologiche utilizzate

L'intera soluzione gira su infrastruttura Microsoft, con una sola eccezione esterna (Telegram), ma può essere tranquillamente integrata tramite teams workflow/PnP PowerShell:

| Componente | Servizio | 
|---|---|
| Raccolta messaggi | Microsoft Graph API
| Generazione script | Azure AI Foundry - Modello GPT-4o mini |
| Sintesi vocale | Azure AI Foundry - Modello gpt-4o-mini-tts |
| Orchestrazione VM | Azure Automation Runbook|
| VM di esecuzione | Azure VM (Windows Server 2025) |
| Consegna | Telegram Bot API |

---

## Cosa ho imparato

Costruire questa soluzione ha confermato alcune cose che sospettavo e me ne ha insegnate di nuove.

**Il prompt engineering fa la differenza.** Non basta dire al modello "genera un podcast". Bisogna definire il numero di battute per tipo di messaggio, l'ordine di priorità, il tono di ogni voce, le transizioni, il riepilogo finale. Ho rilasciato almeno sei versioni del system prompt prima di arrivare a un output che mi convincesse davvero (forse ancora oggi non sono completamente convinto e sicuramente con il passare del tempo lo migliorerò).

**Partire da ciò che già esiste.** Lo script di raccolta dei messaggi è una semplice chiamata API. Azure AI Foundry era già attivo nella mia subscription. L'App Registration era già configurata. Il lavoro vero è stato capire come connettere i pezzi, non costruirli da zero.

---

## Sviluppi futuri

La soluzione è operativa, ma ci sono direzioni in cui la farei evolvere:

- **Bot interattivo**: oggi il bot è passivo — sa solo inviare. Con una Azure Function che gestisce il webhook, potrebbe rispondere a comandi (`/oggi`, `/cerca Copilot`, `/scadenze`) e diventare un vero strumento di consultazione.
- **Alert urgenti**: se arriva un messaggio con severity `high` o tag `Major change`, il bot potrebbe notificare immediatamente senza aspettare il podcast mattutino.

---

## Conclusione

L'idea era semplice: ascoltare gli aggiornamenti di Microsoft 365 in macchina invece di leggerli alla scrivania. Ogni mattina alle 07:25 mi arriva un messaggio su Telegram con un podcast di dieci minuti. Alex e Sara mi aggiornano su tutto quello che è cambiato nel tenant, partendo dai retirement critici e finendo con le nuove funzionalità. Questa soluzione mi permette di arrivare in ufficio già aggiornato.

---
