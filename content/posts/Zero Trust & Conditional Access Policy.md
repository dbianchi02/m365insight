---
title: "Le migliori Conditional Access Policy per proteggere Microsoft 365 con Zero Trust"
date: 2025-12-15T19:00:00+01:00
draft: false
tags: ["identità-sicurezza", "Conditional Access", "Zero-Trust", "Entra ID"]
authors: ["Davide Bianchi"]
description: "Una guida pratica (e “battle-tested”) alle Conditional Access Policy fondamentali: dall’MFA alla protezione token, fino al blocco dell’accesso admin solo da dispositivi compliant/hybrid joined."
---

Le **Conditional Access Policies (CAP)** di **Microsoft Entra ID** sono il “policy engine” di Zero Trust: alla base sono regole **if/then** che decidono se consentire un accesso e **a quali condizioni** (MFA, device conforme, blocco, Terms of Use, controlli di sessione, ecc.).

In un tenant Microsoft 365 moderno, il Conditional Access serve a trasformare i principi di Zero Trust in controlli concreti:

- **Verifica esplicita**: non fidarti solo di password o IP.
- **Minimo privilegio**: proteggi di più dove l’impatto è massimo (admin e portali).
- **Assumi la compromissione**: riduci l’efficacia di token replay e session hijacking.

> Nota importante: le CAP vanno sempre introdotte con **Report-only → Pilot → Rollout** per evitare lockout e impatti non previsti.

---

## Principi di design (che ti evitano incidenti e rollback)

Prima di entrare nel vivo e spiegare la configurazione delle policy, due piccole regole operative che fanno la differenza e che nel mio caso mi hanno sempre aiutato:

1. **Naming standard e policy “pulite”**: una policy = un obiettivo chiaro (debug più semplice).
2. **Break-glass esclusi**: 1–2 account di emergenza *sempre esclusi* da tutte le CAP (e super monitorati).

> Break-glass: sono utenze amministrative dedicate da usare **solo in scenari eccezionali**, ad esempio quando un errore di configurazione (Conditional Access/MFA), un problema di Identity Provider o un incident di sicurezza blocca l’accesso agli amministratori “normali”. L’idea è avere una via di rientro garantita: per questo motivo, in genere, questi account vengono esclusi da tutte le Conditional Access Policy e protetti con misure compensative molto rigide (password lunga e unica, conservazione sicura, monitoraggio/alerting continuo e accesso consentito solo in caso di reale necessità).

---

## Le Conditional Access Policy che consiglio (set completo)

In questa sezione andremo ad approfondire un set di **Conditional Access Policy (CAP)** pensato per coprire i principali pilastri Zero Trust in Microsoft 365: eliminazione dei vettori “legacy”, innalzamento dell’assurance con MFA, protezione degli account privilegiati, remediation guidata dal rischio (Identity Protection) e governance tramite Terms of Use. 

Nel mio caso, sul mio tenant ho scelto di adottare una **naming convention uniforme** con prefisso `[CAP]`, così da riconoscere subito a colpo d’occhio le policy di sicurezza e mantenerle ordinate nel tempo.

Di seguito le policy che analizzeremo e **come le ho denominate nel mio tenant**:

![CAPNaming](/images/posts/security/cap-namingconvention.png)

---

### 1) Block Legacy Authentication

**Obiettivo:** bloccare l’autenticazione legacy (protocolli/client non moderni) che spesso è il primo vettore sfruttato in attacchi di password spraying e brute force.

**Perché è fondamentale:** i protocolli legacy sono più facili da attaccare e possono ridurre l’efficacia dei controlli moderni.

**Implementazione tipica**
- **Users**: All users (escludi break-glass)
- **Target resources**: All cloud apps (o almeno Exchange Online se vuoi partire graduale)
- **Conditions → Client apps**: Other clients / legacy auth
- **Grant**: Block access

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-legacy-authentication)

---

### 2) Entra Token Protection (Session control)

**Obiettivo:** ridurre attacchi di **token replay** e furto sessione, richiedendo che i token accettati siano “legati” al dispositivo/sessione prevista.

**Punto chiave:** non è una “Grant control” come MFA; è hardening della **sessione**. Va pilotata, perché non tutte le combinazioni di app/client hanno lo stesso comportamento.

**Implementazione tipica**
- **Users**: Pilot group (poi estensione)
- **Cloud apps**: Microsoft 365 core (Exchange/SharePoint/Teams) o app critiche
- **Access controls → Session**: **Require token protection for sign-in sessions**
- **Enable**: Report-only → On

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-token-protection)

---

### 3) Require MFA for all users (eccezione Trusted location)

**Obiettivo:** alzare il baseline: se comprometti una password, non deve bastare per entrare.

> Attenzione: “Trusted location” è spesso una scelta di UX. L’IP non dimostra che il **device** sia gestito o sano.

**Implementazione tipica**
- **Users**: All users (escludi break-glass)
- **Locations**: Exclude “Trusted named locations”
- **Grant**: Require MFA *(oppure)* Require authentication strength: **MFA strength**

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-mfa-strength)

---

### 4) Require phishing-resistant MFA for Administrators

**Obiettivo:** proteggere gli account privilegiati con un livello di assurance superiore.

**Best practice moderna:** usa **Authentication Strength: Phishing-resistant MFA** per gli admin, perché riduce la superficie su phishing avanzato (proxy/AiTM).

**Implementazione tipica**
- **Users**: Directory roles (Global Admin + ruoli critici)
- **Cloud apps**: Microsoft Admin Portals (almeno) o All cloud apps
- **Grant**: Require authentication strength → **Phishing-resistant MFA strength**

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-admin-phish-resistant-mfa)

---

### 5) Admins: accesso ai portali admin solo da device compliant/hybrid joined

**Obiettivo:** ridurre drasticamente la probabilità che un furto credenziali admin diventi un takeover del tenant.

**Perché è forte:** anche con credenziali rubate, l’attaccante spesso non ha un device che soddisfa i requisiti di gestione/compliance.

**Implementazione tipica**
- **Users**: Directory roles (admin)
- **Cloud apps**: Microsoft Admin Portals
- **Grant**:
  - Require device to be marked as compliant **OR**
  - Require Microsoft Entra hybrid joined device

> Nota: “compliant” presuppone una strategia MDM/compliance (tipicamente Intune). Senza compliance policy reali, questa CAP non si comporta come ti aspetti.

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-alt-admin-device-compliand-hybrid)

---

### 6) Require remediation for risky sign-in (Sign-in risk)

**Obiettivo:** quando Microsoft Entra rileva un **sign-in rischioso**, richiedi remediation (tipicamente MFA) oppure blocchi se l’utente non è registrato.

**Implementazione tipica**
- **Condition → Sign-in risk**: Medium/High (o High per partire conservativi)
- **Grant**: Require MFA

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-risk-based-sign-in)

---

### 7) Require remediation for risky user (User risk)

**Obiettivo:** se è l’**identità** ad essere considerata compromessa (user risk), la remediation tipica è **password change**.

**Implementazione tipica**
- **Condition → User risk**: High
- **Grant**: Require password change

> Nota licensing: le funzionalità risk-based richiedono **Microsoft Entra ID Protection** (tipicamente Entra ID P2 / Entra Suite).

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-risk-based-user)

---

### 8) CAP Bonus - Force Terms of Use

**Obiettivo:** presentare un documento (PDF) e richiedere l’accettazione prima di concedere accesso a risorse specifiche. È utile per compliance, responsabilità d’uso e “user awareness”.

**Implementazione tipica**
- Crea Terms of Use (PDF, lingua/e, display name)
- Policy CA:
  - **Target resources**: Microsoft Admin Portals (o app sensibili)
  - **Grant**: Require terms of use

* [Configurazione dettagliata – Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-require-terms-of-use)

![ToU](/images/posts/security/tou.png)

---

## Ordine di rollout consigliato (per ridurre rischio e impatto)

1. **Block legacy authentication** (Report-only → On)
2. **MFA per tutti** (pilot + estensione)
3. **Admin: phishing-resistant MFA strength**
4. **Admin portals solo da compliant/hybrid joined**
5. **Risk-based (sign-in risk / user risk)** se hai Entra ID Protection
6. **Token Protection** (pilot, poi ampliamento)
7. **Terms of Use** su portali admin/app sensibili

---

## Conclusione

Conditional Access non è “mettere MFA”: è definire **quanto** e **quando** fidarti, usando segnali diversi e controlli coerenti. Se blocchi l’autenticazione legacy, imponi MFA (meglio se governata con Authentication Strengths), introduci remediation basata sul rischio e aggiungi hardening di sessione con Token Protection, stai già coprendo la maggior parte dei pattern di attacco moderni.

Il vero salto di maturità, però, arriva quando separi nettamente l’accesso “utente” dall’accesso “amministrativo”: gli admin non devono solo autenticarsi meglio, devono amministrare **da dispositivi gestiti e conformi**. Limitare l’accesso ai Microsoft Admin Portals a device **compliant o hybrid joined** è una delle misure più efficaci per impedire che un furto credenziali si trasformi in compromissione completa del tenant.

Infine, la sicurezza non si misura dal numero di policy attive, ma dal modo in cui le governi: **Report-only, pilota, monitoraggio nei sign-in logs e disciplina sulle eccezioni**. Conditional Access è un controllo continuo: cambia insieme al tuo modo di lavorare, alle tue app e alle tecniche d’attacco.