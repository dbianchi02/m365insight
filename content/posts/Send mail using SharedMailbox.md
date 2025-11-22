---
title: "Inviare una mail da una shared mailbox con Microsoft Graph (app-only)"
date: 2025-11-22T10:00:00+02:00
draft: false
tags: ["automazione-it", "Microsoft Graph", "Exchange Online", "PowerShell"]
description: "Come configurare un’app in Entra ID e i permessi necessari per inviare email da una shared mailbox tramite Microsoft Graph in modalità app-only."
---

## Introduzione

Nelle automazioni quotidiane (backup, job schedulati, script di monitoraggio) è spesso necessario inviare una mail di notifica da una casella condivisa del tenant, ad esempio `alert@azienda.it`. In passato questa esigenza veniva risolta con SMTP e autenticazione di base; oggi, con la dismissione della basic auth in Exchange Online, l’approccio consigliato è usare **Microsoft Graph** in modalità **app-only** (flusso client credentials).

In questo articolo vedremo come preparare l’ambiente lato tenant per inviare email da una **shared mailbox** utilizzando Graph. In particolare ci concentreremo su:

* registrazione dell’app in Entra ID;
* configurazione dei permessi Microsoft Graph;
* limitazione dell’app alla sola shared mailbox tramite **Application Access Policy**;
* recupero dei parametri da usare nello script PowerShell (che aggiungerai tu nella sezione finale).

---

### Scenario: notifiche automatiche da una shared mailbox

Lo scenario tipico è il seguente:

* uno script PowerShell viene eseguito da un server (on-prem o VM in cloud);
* al termine di un’operazione (es. backup riuscito o fallito) lo script deve inviare una mail di notifica;
* la mail viene inviata da una casella tecnica condivisa, es. `alert@azienda.it`, verso uno o più destinatari interni/esterni.

Invece di usare le credenziali di un utente reale, lo script utilizza un’identità applicativa (App Registration) per ottenere un **access token** e chiamare l’endpoint Graph `sendMail` sulla shared mailbox.

---

### Perché usare Microsoft Graph app-only?

Usare Graph in modalità app-only presenta diversi vantaggi:

* **Niente credenziali utente**: lo script non dipende da un account umano che può essere disabilitato, scadere o richiedere MFA.
* **Modello di permessi chiaro**: i permessi sono definiti a livello di applicazione (`Mail.Send` in Application Permissions) e possono essere ulteriormente ristetti a un set di mailbox specifiche.
* **Compliance con le best practice Microsoft**: si allinea alle raccomandazioni di sicurezza per Exchange Online e M365.

Al tempo stesso, un’app con `Mail.Send` application potrebbe teoricamente inviare mail da **qualsiasi mailbox** nel tenant. Per questo è fondamentale aggiungere un livello di controllo ulteriore tramite Application Access Policy, che vedremo più avanti.

---

### Prerequisiti

Prima di iniziare, verifica di avere:

* Una **shared mailbox** esistente (es. `alert@azienda.it`) o la possibilità di crearne una nuova.
* Permessi adeguati:
  * ruolo **Application Administrator** o **Global Administrator** per la parte Entra ID / Graph;
  * ruolo **Exchange Administrator** per configurare le Application Access Policy.
* Accesso al portale **Entra ID** (ex Azure AD) e **Exchange admin center**.
* PowerShell con moduli installati, disponibile sulla macchina da cui gireranno gli script.

---

### Panoramica della soluzione

La soluzione si basa su quattro elementi chiave:

1. **Shared mailbox**: la casella “tecnica” da cui partiranno le notifiche (es. `alert@azienda.it`).
2. **App Registration in Entra ID**: rappresenta l’identità applicativa che lo script userà per autenticarsi.
3. **Permessi Microsoft Graph (Mail.Send – Application)**: consentono all’app di inviare email.
4. **Application Access Policy**: limita l’app solo alla shared mailbox designata, impedendo l’abuso su altre caselle.

Lo script PowerShell:

* richiede un token OAuth2 con grant type `client_credentials`;
* chiama l’endpoint `https://graph.microsoft.com/v1.0/users/{sharedMailbox}/sendMail`;
* invia la mail di notifica usando la shared mailbox come mittente.

---

## Procedura passo-passo

### 1. Creare (o identificare) la shared mailbox

Se hai già una casella condivisa da usare come mittente, puoi riferirti direttamente a quell’indirizzo. In caso contrario:

1. Accedi all’**Exchange admin center**.
2. Vai su **Recipients** → **Shared**.
3. Clicca su **Add a shared mailbox**.
4. Compila:
   * **Name**: es. `Alert Backup`.
   * **Email**: es. `alert@azienda.it`.
5. Completa la creazione della mailbox.

Questa sarà l’identità “da cui” Graph invierà le email nel tuo script:

![Shared](/images/posts/automazione/shared.png)

---

### 2. Registrare l’applicazione in Entra ID

L’app che registrerai in Entra ID è l’identità tecnica usata dallo script per autenticarsi su Microsoft Graph.

1. Vai al portale **Entra ID** → **App registrations**.
2. Clicca su **New registration**.
3. Imposta:
   * **Name**: ad esempio `Graph-SharedMailbox-Alert`.
   * **Supported account types**: in genere **Single tenant** è sufficiente (solo il tuo tenant).
   * **Redirect URI**: non necessario per il flusso client credentials, puoi lasciarlo vuoto.
4. Clicca su **Register**.

Dopo la creazione, prendi nota di:

* **Application (client) ID** → che userai come `clientId` nello script.
* **Directory (tenant) ID** → che userai come `tenantId`.

---

### 3. Creare il client secret

Per autenticarsi in modalità app-only, lo script avrà bisogno di un **client secret** (oppure di un certificato; qui ci concentriamo sul secret).

1. Nella pagina dell’app, vai su **Certificates & secrets**.
2. Nella sezione **Client secrets**, clicca **New client secret**.
3. Inserisci una descrizione (es. `AlertBackup-Prod`), scegli una scadenza adeguata.
4. Clicca **Add**.
5. Copia **subito** il valore del secret: è quello che userai nello script come `clientSecret`.

> Se perdi il valore del secret, non potrai recuperarlo: dovrai crearne uno nuovo.

---

### 4. Aggiungere i permessi Microsoft Graph (Mail.Send – Application)

Ora devi concedere all’app il permesso di inviare email tramite Graph.

1. Nella pagina dell’app, vai su **API permissions**.
2. Clicca **Add a permission** → **Microsoft Graph**.
3. Seleziona **Application permissions** (non Delegated).
4. Cerca e aggiungi:
   * `Mail.Send` (Application).
5. Clicca su **Add permissions**.
6. Una volta aggiunto, clicca su **Grant admin consent** per il tenant.

Senza l’admin consent, l’app potrebbe ottenere un token, ma le chiamate a `sendMail` fallirebbero per mancanza di autorizzazioni effettive.

---

### 5. Limitare l’app alla sola shared mailbox (Application Access Policy)

Fin qui, l’app ha potenzialmente la capacità di inviare email da qualunque mailbox del tenant. Per ridurre il rischio, è buona pratica limitare esplicitamente il suo ambito tramite una **Application Access Policy** in Exchange Online.

L’idea è:

* creare un gruppo che contiene **solo** le mailbox da cui l’app è autorizzata a operare (ad esempio solo `alert@azienda.it`);
* legare l’app a quel gruppo con una policy `RestrictAccess`.

#### 5.1 Creare il gruppo di ambito

Puoi usare un **mail-enabled security group** o un gruppo Microsoft 365 dedicato, ad esempio:

* Nome: `Graph-Alert-Mailboxes`
* Alias: `Graph-Alert-Mailboxes@azienda.it`

Aggiungi come membro la shared mailbox `alert@azienda.it`.

#### 5.2 Creare l’Application Access Policy

Da una sessione PowerShell con modulo Exchange Online:

```powershell
Install-Module ExchangeOnlineManagement -Scope AllUsers
Import-Module ExchangeOnlineManagement

Connect-ExchangeOnline -UserPrincipalName admin@azienda.it

# Application (client) ID dell’app registrata in Entra ID
$appId = "INSERISCI-CLIENT-ID-APP"

# Gruppo che contiene le mailbox autorizzate (mail-enabled security group)
$policyScopeGroup = "Graph-Alert-Mailboxes@azienda.it"

New-ApplicationAccessPolicy `
    -AppId $appId `
    -PolicyScopeGroupId $policyScopeGroup `
    -AccessRight RestrictAccess `
    -Description "Consente all'app di usare solo le mailbox del gruppo Graph-Alert-Mailboxes"

# Per verificare il comportamento corretto:
Test-ApplicationAccessPolicy -AppId $appId -Identity alert@azienda.it   # atteso: Granted
Test-ApplicationAccessPolicy -AppId $appId -Identity ceo@azienda.it     # atteso: Denied

```

### 6. Recuperare i parametri per lo script PowerShell

A questo punto hai tutti i valori necessari per lo script che invierà la mail via Graph:

- `clientId` → **Application (client) ID** dell’app.  
- `tenantId` → **Directory (tenant) ID** del tenant M365.  
- `clientSecret` → **client secret** creato in precedenza.  
- `MailSender` → indirizzo della shared mailbox, ad esempio `alert@azienda.it`.  
- **URL token** → `https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token`  
- **Scope** → `https://graph.microsoft.com/.default` (per usare i permessi Application configurati sull’app).  
- **Endpoint di invio mail** → `https://graph.microsoft.com/v1.0/users/{MailSender}/sendMail`

 **Esempio di script PowerShell**

``` powershell
<# 
********************************************************************************************************************
INFO

    Author          :    Bianchi Davide
    UserMail        :    *****@dbianchi.it
    Company         :    
    Current Version :    1.2
    Created         :    08/06/2023
    Last Updated    :    13/06/2023

********************************************************************************************************************
DESCRIPTION

    This script connects to Microsoft Graph and sends an email using a Shared Mailbox. In this specific case, 
    it simulates sending an email to notify of the successful execution of a backup.

********************************************************************************************************************
#>

#ID for Graph Token Generation
$clientID = "inserire credenziali"
$Clientsecret = "inserire credenziali"
$tenantID = "inserire credenziali"

#SharedMailBox (from)
$MailSender = "inserire indirizzo della shared"

#Connect to GRAPH API
$tokenBody = @{
    Grant_Type    = "client_credentials"
    Scope         = "https://graph.microsoft.com/.default"
    Client_Id     = $clientId
    Client_Secret = $clientSecret
}
$tokenResponse = Invoke-RestMethod -Uri "https://login.microsoftonline.com/$tenantID/oauth2/v2.0/token" -Method POST -Body $tokenBody
$headers = @{
    "Authorization" = "Bearer $($tokenResponse.access_token)"
    "Content-type"  = "application/json"
}

#Send Mail    
$URLsend = "https://graph.microsoft.com/v1.0/users/$MailSender/sendMail"
$BodyJsonsend = @"
                    {
                        "message": {
                          "subject": "Stato esecuzione Backup",
                          "body": {
                            "contentType": "HTML",
                            "content": "Backup avvenuto con successo, <br>
                            Buona Giornata <br>
                            
                            "
                          },
                          "toRecipients": [
                            {
                              "emailAddress": {
                                "address": "indirizzo mail destinatario 1"
                              }
                            },
                            {
                              "emailAddress": {
                                "address": "indirizzo mail destinatario 2"
                              }
                            },
                            {
                              "emailAddress": {
                                "address": "indirizzo mail destinatario 3"
                              }
                            }
                          ]
                        },
                        "saveToSentItems": "false"
                      }
"@

Invoke-RestMethod -Method POST -Uri $URLsend -Headers $headers -Body $BodyJsonsend

```
Una volta schedulato ed eseguito lo Script, arriverà la mail agli indirizzi specificati: 
![mail](/images/posts/automazione/mailbackup.png)
---

### Punti critici e buone pratiche

#### Gestione del secret

Evita di lasciare il `clientSecret` in chiaro nei file di script. Quando possibile, valuta l’uso di un **certificato** o di un secret archiviato in **Key Vault**, in un password manager o in un file protetto a livello di sistema.

#### Ambito dei permessi

`Mail.Send` in Application è un permesso potente: se dimentichi di configurare correttamente la **Application Access Policy**, l’app può operare su molte più mailbox del previsto. Vale la pena verificare con `Test-ApplicationAccessPolicy` e con test mirati prima di andare in produzione.

#### Logging e tracciabilità

Anche se la mailbox è tecnica, conserva i log delle chiamate e, se necessario, abilita la conservazione degli elementi inviati (`saveToSentItems`), almeno negli ambienti di test o per i primi rilasci in produzione.

---

## Conclusione

Configurare l’invio di email da una shared mailbox tramite Microsoft Graph in modalità app-only permette di automatizzare notifiche e alert in modo sicuro e moderno, senza dipendere da account utente o da protocolli legacy.  
Con una corretta **App Registration**, permessi Graph ben definiti e una **Application Access Policy** restrittiva, puoi ottenere un canale di notifica affidabile, governato e allineato alle best practice di sicurezza di Microsoft 365. Potrai integrare lo script PowerShell completo che utilizza questi parametri per inviare le mail al termine dei tuoi job di backup o di monitoraggio.