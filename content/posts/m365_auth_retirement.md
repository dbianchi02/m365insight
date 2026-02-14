---
title: "Retirement IDCRL in SharePoint Online: migrazione all'autenticazione moderna"
date: 2026-01-24T12:00:00+02:00
draft: false
tags: ["identità-sicurezza", "SharePoint", "Authentication", "IDCRL", "OAuth", "Microsoft 365"]
authors: ["Davide Bianchi"]
description: "Microsoft rimuoverà l'autenticazione IDCRL da SharePoint Online entro maggio 2026. Guida completa alla migrazione verso OAuth e MSAL con timeline, verifiche e soluzioni pratiche."
---

## Introduzione

Microsoft ha annunciato la dismissione dell'autenticazione IDCRL (Identity Client Run Time Library) in SharePoint Online, un ulteriore passo verso l'adozione esclusiva di protocolli di autenticazione moderni come OAuth 2.0 e OpenID Connect. Dal 2018, l'accesso standard a SharePoint e OneDrive si basa già su questi protocolli sicuri, ma alcune chiamate API continuavano a supportare IDCRL per retrocompatibilità.

Questo cambiamento richiede un intervento proattivo degli amministratori IT per garantire la continuità operativa di applicazioni e script che ancora utilizzano il protocollo legacy.

![IDCRL-Retirement](/images/posts/security/idcrl-auth-blog-post.png)

## Cos'è l'autenticazione IDCRL

L'IDCRL (Identity Client Run Time Library) è un protocollo di autenticazione legacy utilizzato per accedere a SharePoint Online e OneDrive for Business. Microsoft sta eliminando completamente questo supporto legacy per garantire che tutte le chiamate di autenticazione utilizzino esclusivamente i protocolli moderni.

### Perché Microsoft rimuove IDCRL

L'autenticazione IDCRL presenta vulnerabilità di sicurezza che la rendono inadeguata agli standard di protezione attuali. I protocolli moderni offrono:

- **Migliore protezione** contro attacchi di phishing e furto di credenziali
- **Supporto nativo per MFA** (Multi-Factor Authentication)
- **Token di accesso** con scadenza limitata invece di credenziali persistenti
- **Controllo granulare** sui permessi delle applicazioni

## Timeline del retirement

Microsoft ha definito una roadmap precisa:

| Data | Evento | Azione |
|------|--------|--------|
| **31 gennaio 2026** | Blocco di default | Autenticazione legacy bloccata (posticipabile via PowerShell) |
| **30 aprile 2026** | Scadenza posticipo | Ultima possibilità di usare IDCRL con estensione temporanea |
| **1 maggio 2026** | Blocco definitivo | Autenticazione IDCRL completamente disabilitata, senza possibilità di riabilitazione |

### Posticipare temporaneamente il retirement

Gli amministratori possono riabilitare temporaneamente IDCRL fino al 30 aprile 2026:

```powershell
# Abilitare temporaneamente IDCRL
Set-SPOTenant -AllowLegacyAuthProtocolsEnabledSetting $true -LegacyAuthProtocolsEnabled $true

# Verificare lo stato corrente
Get-SPOTenant | Select AllowLegacyAuthProtocolsEnabledSetting, LegacyAuthProtocolsEnabled
```

**⚠️ Attenzione**: Questa è una soluzione temporanea. Dal 1° maggio 2026 non sarà più possibile riabilitare l'autenticazione legacy.

## Verificare l'utilizzo di IDCRL

Microsoft fornisce anche soluzioni di auditing per identificare se la tua organizzazione utilizza IDCRL.

### Accesso agli strumenti di monitoraggio tramite Microsoft Purview

**Ruoli con accesso:**
- Global Administrator
- Compliance Administrator
- Purview Compliance Administrator
- Role Management Role
- Collection Admin, Data Curator, Data Reader

**Procedura:**

1. Accedi a [Microsoft Purview](https://compliance.microsoft.com) o [Purview Portal](https://purview.microsoft.com)
2. Naviga su **Audit** nel pannello laterale
3. In "Activities – operations name" seleziona **"IDCRLSuccessSignIn"**
4. Applica filtri aggiuntivi se necessario
5. Clicca su **Search**

![IDCRL-audit](/images/posts/security/audit-purview-IDCRL.png)

I risultati mostreranno tutte le chiamate IDCRL, permettendoti di identificare quali applicazioni o script necessitano aggiornamento.

## Quali chiamate utilizzano IDCRL

Esistono **due categorie** di chiamate IDCRL:

### Categoria 1: Utilizzo della libreria SharePointOnlineCredentials

Qualsiasi utilizzo della libreria `SharePointOnlineCredentials` nel codice indica chiamate IDCRL, poiché questa libreria utilizza il protocollo IDCRL sotto il cofano.

```csharp
// Esempio di codice che usa IDCRL (DA AGGIORNARE)
SharePointOnlineCredentials credentials = new SharePointOnlineCredentials(username, password);
```

### Categoria 2: Chiamate dirette agli endpoint IDCRL

Tutte le chiamate a questi endpoint utilizzano il protocollo IDCRL:
- `https://login.microsoftonline.com/rst2.srf` (per ottenere SAML BinarySecurityToken)
- `https://TENANT.sharepoint.com/_vti_bin/idcrl.svc` (per scambiare token con cookie SPOIDCRL)

## Soluzioni per la migrazione

### Opzione 1: Microsoft Authentication Library (MSAL) - Soluzione raccomandata

MSAL è la soluzione primaria raccomandata da Microsoft per acquisire token OAuth che funzionano sia per chiamate di Categoria 1 che Categoria 2.

**Risorse tecniche:**
- [Using modern authentication with CSOM for .NET Standard](https://learn.microsoft.com/en-us/sharepoint/dev/sp-add-ins/using-csom-for-dotnet-standard#using-modern-authentication-with-csom-for-net-standard)
- [Configuring an application in Azure AD](https://learn.microsoft.com/en-us/sharepoint/dev/sp-add-ins/using-csom-for-dotnet-standard#configuring-an-application-in-azure-ad)

**Documentazione OAuth e MSAL:**
- [What Is OAuth? | Microsoft Security](https://www.microsoft.com/en-us/security/business/security-101/what-is-oauth)
- [OAuth 2.0 authorization code flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- [Overview of MSAL](https://learn.microsoft.com/en-us/entra/identity-platform/msal-overview)

### Opzione 2: Aggiornamento del pacchetto NuGet CSOM

Microsoft ha rilasciato una versione aggiornata del pacchetto NuGet che permette la transizione da IDCRL a OAuth.

**Download:** [Microsoft.SharePointOnline.CSOM su NuGet](https://www.nuget.org/packages/Microsoft.SharePointOnline.CSOM/)

#### Modifiche necessarie nel codice

**Metodo deprecato `GetAuthenticationCookie`:**

Il metodo viene sostituito da `AcquireTokenAsync` che acquisisce token OAuth.

**Gestione della Multi-Factor Authentication:**

Se la Multi-Factor Authentication è abilitata, è necessario passare un nuovo parametro al costruttore dell'oggetto SharePointOnlineCredentials. 
Ecco come va modificato il codice:

```csharp
SharePointOnlineCredentials credentials = new SharePointOnlineCredentials(
    username, 
    password, 
    useModernAuth: true, 
    interactiveAuth: true  // Abilita autenticazione interattiva per MFA
);
```

**App-Only Authentication (senza interazione utente):**

Per scenari automatizzati senza intervento utente, configura App-Only Authentication durante la registrazione dell'app in Microsoft Entra.

## Checklist per gli amministratori

- [ ] Verificare chiamate IDCRL tramite Microsoft Purview Audit
- [ ] Identificare tutte le applicazioni che usano `SharePointOnlineCredentials`
- [ ] Pianificare migrazione a MSAL/OAuth per ogni applicazione
- [ ] Testare implementazioni in ambiente dev/staging
- [ ] Se necessario, abilitare posticipo temporaneo via PowerShell
- [ ] Completare migrazione entro il 30 aprile 2026

## Conclusioni

La rimozione dell'autenticazione IDCRL rappresenta un passo necessario per migliorare la sicurezza di SharePoint Online. Gli amministratori hanno tempo fino al 30 aprile 2026 per completare la migrazione, ma è consigliabile iniziare subito l'assessment e la pianificazione.

Il posticipo temporaneo via PowerShell offre un buffer di sicurezza, ma ricorda che dal 1° maggio 2026 non ci saranno più alternative: l'autenticazione moderna diventerà obbligatoria.

---

**Riferimenti:**
- [Migrating from IDCRL authentication to modern authentication in SharePoint](https://devblogs.microsoft.com/microsoft365dev/migrating-from-idcrl-authentication-to-modern-authentication-in-sharepoint/)