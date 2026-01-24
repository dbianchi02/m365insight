---
title: "Exchange Online: dismissione Basic Auth per SMTP AUTH entro aprile 2026"
date: 2026-01-24T11:00:00+02:00
draft: false
tags: ["identità-sicurezza", "Exchange Online", "Security", "SMTP", "Authentication", "OAuth", "Microsoft 365"]
authors: ["Davide Bianchi"]
description: "Microsoft rimuove la Basic Authentication per SMTP AUTH in Exchange Online da marzo 2026. Guida alle alternative: OAuth, High Volume Email, Azure Communication Services e soluzioni ibride."
---

## Introduzione

Exchange Online completerà ad aprile 2026 il processo di dismissione della Basic Authentication iniziato nel 2019. L'unica eccezione rimasta, Client Submission (SMTP AUTH), verrà finalmente migrata ai protocolli di autenticazione moderni.

Questo cambiamento impatterà applicazioni, dispositivi multifunzione, sistemi di monitoraggio e tutti i servizi che attualmente inviano email tramite SMTP AUTH con credenziali in chiaro. Gli amministratori devono agire ora per evitare interruzioni di servizio.

![SMTP-BasicAuth-Retirement](/images/posts/security/Retirement-SMPT-BasicAuth.png)

## Cos'è la Basic Authentication per SMTP AUTH

La Basic Authentication per Client Submission (SMTP AUTH) è un metodo di autenticazione legacy che trasmette username e password sulla rete. Sebbene normalmente protetto da TLS, questo metodo presenta vulnerabilità significative rispetto ai protocolli moderni.

### Perché Microsoft rimuove la Basic Auth

Dal 2019, Exchange Online ha lavorato alla dismissione della Basic Authentication, completata nel 2022 per tutti i protocolli eccetto SMTP AUTH. Ora Microsoft rimuove anche quest'ultima eccezione per:

- **Eliminare vulnerabilità** a furto di credenziali, phishing e attacchi brute force
- **Impedire trasmissione** di credenziali facilmente intercettabili
- **Eliminare la mancanza** di supporto nativo per Multi-Factor Authentication
- **Allinearsi agli standard** di sicurezza moderni con OAuth 2.0

## Timeline del retirement

Microsoft ha aggiornato le tempistiche nel giugno 2025:

| Data | Evento | Dettaglio |
|------|--------|-----------|
| **Metà ottobre 2024** | Report aggiornato | EAC mostra se viene usata Basic Auth o OAuth |
| **1 marzo 2026** | Inizio rollout | Rifiuto graduale delle richieste Basic Auth (partenza con piccola percentuale) |
| **30 aprile 2026** | Completamento | 100% delle richieste Basic Auth rifiutate |

### Endpoint interessati

- `smtp.office365.com`
- `smtp-legacy.office365.com`

### Messaggio di errore post-dismissione

```
550 5.7.30 Basic authentication is not supported for Client Submission.
```

## Verificare l'utilizzo di Basic Auth

### Report SMTP AUTH Clients nell'Exchange Admin Center

1. Accedi all'[Exchange admin center](https://admin.exchange.microsoft.com)
2. Naviga su **Reports** > **Mail flow**
3. Apri il report **"SMTP AUTH clients submission report"**
4. Controlla la colonna **"Authentication Protocol"**:
   - **"Modern Authentication"** = OAuth (non sarà impattato) ✅
   - **"Basic Authentication"** = richiede migrazione ⚠️

Questo report ti permette di identificare esattamente quali client, applicazioni o dispositivi necessitano aggiornamento.

## Soluzioni per la migrazione

Microsoft offre diverse opzioni a seconda del caso d'uso:

### Opzione 1: OAuth con SMTP AUTH (Raccomandata)

**Soluzione migliore** se il tuo client o applicazione supporta OAuth.

**Guida implementazione:**
- [Authenticate an IMAP, POP or SMTP connection using OAuth](https://learn.microsoft.com/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth)

**✅ Vantaggi:**
- Massima sicurezza con token a breve durata
- Supporto nativo per MFA
- Nessun costo aggiuntivo
- Soluzione a lungo termine

**📋 Requisiti:**
- Registrazione applicazione in Microsoft Entra ID
- Implementazione flusso OAuth 2.0
- Configurazione permessi appropriati

### Opzione 2: High Volume Email for Microsoft 365 (HVE)

**Caso d'uso:** Solo email **interne** al tenant (tra utenti della stessa organizzazione).

High Volume Email (attualmente in Public Preview) è progettato per applicazioni line-of-business e submission ad alto volume che inviano messaggi interni in Exchange Online.

**✅ Vantaggi:**
- Supporto continuo per Basic Authentication
- Ottimizzato per alto volume
- Incluso in Microsoft 365
- Alternativa per decommissionare server on-premises

**⚠️ Limitazioni:**
- Solo per email interne al tenant
- Richiede onboarding e configurazione

**Guida:** [Manage high volume emails for Microsoft 365](https://learn.microsoft.com/Exchange/mail-flow-best-practices/high-volume-mails-m365)

### Opzione 3: Azure Communication Services for Email

**Caso d'uso:** Email **interne ed esterne** al tenant.

Piattaforma centralizzata per gestire email in uscita per comunicazioni B2C, con supporto SMTP e analytics avanzate.

**✅ Vantaggi:**
- Supporto destinatari interni ed esterni
- Piattaforma gestita con insights
- Maggiore controllo sulle comunicazioni
- Scalabilità enterprise

**💰 Considerazioni:**
- Servizio Azure separato (pay-per-use)
- Richiede configurazione in Azure Portal
- Ideale per scenari B2C e transazionali

![ACS-FlowChart](/images/posts/security/Exchange-Online-Interacting-with-Azure-Communication-Services-Email.png)

**Guida:** [Email SMTP as service overview in Azure Communication Services](https://learn.microsoft.com/azure/communication-services/concepts/email/email-smtp-overview)

### Opzione 4: Exchange Server On-Premises (Ibrido)

**Caso d'uso:** Organizzazioni con Exchange Server on-premises in configurazione ibrida.

Autenticazione con Exchange on-premises o configurazione di Receive connector con anonymous relay.

**Configurazione:** [Allow anonymous relay on Exchange servers](https://learn.microsoft.com/en-us/exchange/mail-flow/connectors/allow-anonymous-relay?view=exchserver-2019)

**⚠️ Considerazioni:**
- Richiede infrastruttura on-premises
- Manutenzione e gestione server locale
- Soluzione temporanea se pianifichi migrazione cloud completa

## Confronto delle soluzioni

| Soluzione | Destinatari | Basic Auth | Costi | Complessità | Raccomandato |
|-----------|-------------|------------|-------|-------------|--------------|
| OAuth con SMTP AUTH | Interni/Esterni | No | Nessuno | Media | ⭐⭐⭐⭐⭐ |
| High Volume Email | Solo interni | Sì | Incluso M365 | Bassa | ⭐⭐⭐⭐ |
| Azure Comm. Services | Interni/Esterni | Sì | Pay-per-use | Media | ⭐⭐⭐ |
| Exchange On-Premises | Interni/Esterni | Sì | Infrastruttura | Alta | ⭐⭐ |

## FAQ - Domande frequenti

**❓ Perché questo cambiamento?**  
Per rafforzare la protezione del servizio e dei dati dagli crescenti rischi della Basic Authentication.

**❓ Posso riabilitare Basic Auth dopo marzo 2026?**  
No. Non sarà possibile e il supporto Microsoft non può concedere eccezioni.

**❓ Esistono eccezioni?**  
No. Microsoft non offre eccezioni e la Basic Auth sarà permanentemente disabilitata.

**❓ Gli endpoint smtp-mail.outlook.com o outlook.office365.com sono impattati?**  
No. Questi endpoint sono per applicazioni OAuth 2.0 e non saranno impattati.

**❓ Il mio dispositivo multifunzione funzionerà?**  
Verifica con il produttore se supporta OAuth 2.0. Altrimenti valuta HVE (solo email interne) o Azure Communication Services.

**❓ Cosa succede se non faccio nulla?**  
Dal 1° marzo 2026 inizierai a vedere rifiuti graduali, fino al 100% entro il 30 aprile 2026.

## Checklist per gli amministratori

- [ ] Verificare utilizzo Basic Auth tramite report SMTP AUTH nell'EAC
- [ ] Identificare tutti i dispositivi/applicazioni che inviano via SMTP AUTH
- [ ] Scegliere la soluzione appropriata per ogni caso d'uso
- [ ] Implementare e testare la soluzione scelta in ambiente di test
- [ ] Monitorare report per verificare passaggio a "Modern Authentication"
- [ ] Completare migrazione entro febbraio 2026

## Conclusioni

La dismissione della Basic Authentication per SMTP AUTH è l'ultimo tassello del processo iniziato nel 2019. Gli amministratori hanno tempo fino ad aprile 2026, ma è fondamentale agire ora per:

1. **Identificare** tutti i sistemi che usano Basic Auth
2. **Valutare** la soluzione più adatta (OAuth è la scelta migliore quando possibile)
3. **Testare** le implementazioni
4. **Migrare** prima dell'inizio del rollout graduale a marzo 2026

Non esistono eccezioni o possibilità di posticipo: la pianificazione tempestiva è essenziale per evitare interruzioni di servizio.

---

**Riferimenti:**
- [Exchange Online to retire Basic auth for Client Submission (SMTP AUTH)](https://techcommunity.microsoft.com/blog/exchange/exchange-online-to-retire-basic-auth-for-client-submission-smtp-auth/4114750)
- [Deprecation of Basic authentication in Exchange Online](https://learn.microsoft.com/exchange/clients-and-mobile-in-exchange-online/deprecation-of-basic-authentication-exchange-online)
