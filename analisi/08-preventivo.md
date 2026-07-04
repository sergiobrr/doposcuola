---
title: Preventivo
nav_order: 9
parent: Analisi
---

# Preventivo — App Doposcuola

← [Indice](./README.md)

**Team:** 1 sviluppatore senior (10+ anni) + Claude Code come co-sviluppatore AI  
**Disponibilità:** 4h sabato + 4h domenica = **8h/weekend**  
**Tariffa:** €25/h (tasse e oneri inclusi)  
**Data stima:** giugno 2026

---

## Nota Metodologica

Le ore riportate sono le ore **effettive** del developer umano. Claude Code contribuisce come pair-programmer AI: genera boilerplate, suggerisce architettura, scrive test, fa code review inline. Il risparmio stimato rispetto al lavoro solo è **del 25-30%** sui task ripetitivi (scaffolding, componenti UI, Cloud Functions standard) e **10-15%** sui task complessi (logica business, Firestore Rules, Stripe).

La colonna "Solo" serve come riferimento; il preventivo reale è la colonna **"Con Claude"**.

---

## Dettaglio Fasi

### Fase 1 — Setup e Architettura di Base

| Task | Solo | Con Claude |
|------|------|-----------|
| Scaffold Ionic + Angular 21, Capacitor (iOS + Android) | 3h | 2h |
| Configurazione Firebase: 3 ambienti (dev / staging / prod) + regole base Firestore | 4h | 3h |
| Struttura cartelle, routing lazy-load, environment files | 3h | 2h |
| Guards (AuthGuard, RoleGuard per teacher/parent/student) | 2h | 1.5h |
| NgRx SignalStore: AuthStore, NotificheStore, ComunicazioniStore | 4h | 3h |
| Modelli TypeScript (interfacce da `04-modello-dati.md`) | 2h | 1h |
| **Totale Fase 1** | **18h** | **12.5h** |

---

### Fase 2 — Autenticazione

| Task | Solo | Con Claude |
|------|------|-----------|
| Firebase Auth: login, registrazione genitore, recupero password | 4h | 3h |
| Cloud Function `setUserRole` (Custom Claim al primo login) | 2h | 1.5h |
| Schermate: Login, Registrazione, Forgot Password | 5h | 3.5h |
| Onboarding notifiche push (consenso + registrazione token FCM) | 3h | 2h |
| **Totale Fase 2** | **14h** | **10h** |

---

### Fase 3 — Backend Cloud Functions

| Task | Solo | Con Claude |
|------|------|-----------|
| Stripe: creazione Customer + Payment Intent (pagamento annuale) | 4h | 3h |
| Stripe: Subscriptions (mensile / trimestrale) | 3h | 2h |
| Stripe: Webhook handler (payment succeeded/failed, subscription deleted) | 5h | 3.5h |
| Stripe: Customer Portal session | 1h | 0.5h |
| Sistema notifiche: engine template con variabili `{{...}}` + invio FCM | 6h | 4h |
| Cron jobs: `evento_domani`, `evento_tra_3_giorni`, `pagamento_in_scadenza`, `pagamento_scaduto` | 5h | 3.5h |
| Trigger: `onAssenzaSegnalata`, `onAvvisoCreato`, `onChiusuraCreata`, `onEventoCreato` | 4h | 3h |
| Firestore Security Rules complete (tutte le collezioni) | 5h | 3.5h |
| **Totale Fase 3** | **33h** | **23h** |

---

### Fase 4 — Area Insegnante

| Task | Solo | Con Claude |
|------|------|-----------|
| Dashboard (riepilogo, contatori, quick actions) | 5h | 3.5h |
| Registro Presenze (check-in giornaliero, navigazione date, riepilogo mensile) | 10h | 7h |
| Gestione Iscrizioni (lista con filtri, dettaglio, approva/rifiuta) | 7h | 5h |
| Gestione Studenti — lista + ricerca | 3h | 2h |
| Profilo Studente (dati, orario editor, permessi, note private, storico) | 12h | 8.5h |
| Calendario — vista mensile/settimanale con libreria | 8h | 6h |
| Calendario — CRUD eventi (assegna a studenti, tipo, materia) | 5h | 3.5h |
| Calendario — eventi ricorrenti (serie, generazione occorrenze, override singola) | 8h | 6h |
| Gestione Pagamenti (lista, filtri, export CSV) | 5h | 3.5h |
| Comunicazioni — lista thread + badge | 4h | 3h |
| Chat real-time (Firestore listener, invio, allegati fase 2) | 8h | 6h |
| Impostazioni: Template Notifiche (CRUD con variabili) | 5h | 3.5h |
| Impostazioni: Chiusure/Eccezioni calendario | 3h | 2h |
| Impostazioni: Profilo insegnante | 2h | 1.5h |
| **Totale Fase 4** | **85h** | **60.5h** |

---

### Fase 5 — Area Genitore

| Task | Solo | Con Claude |
|------|------|-----------|
| Lista attività (landing post-login, card serieEventi, posti disponibili) | 4h | 3h |
| Calendario attività (occorrenze, slot, modale iscrizione figlio) | 7h | 5h |
| Gestione figli (lista profili, aggiungi figlio, calendario figlio) | 4h | 3h |
| Pagamenti (lista, Stripe Payment Sheet nativo, Customer Portal) | 7h | 5h |
| Checkout: pagina descrizione servizio + prosegui a Stripe | 2h | 1h |
| Comunicazioni — lista thread + risposta (riusa componenti chat) | 4h | 2.5h |
| Profilo genitore (dati, cambio password, consenso notifiche) | 2h | 1.5h |
| **Totale Fase 5** | **30h** | **20.5h** |

---

### Fase 6 — Testing e Rifinitura

| Task | Solo | Con Claude |
|------|------|-----------|
| Unit test: Store NgRx (AuthStore, NotificheStore, ComunicazioniStore) | 5h | 3.5h |
| Unit test: Cloud Functions principali (Stripe webhook, notifiche) | 4h | 3h |
| Test manuale su dispositivo fisico iOS | 3h | 3h |
| Test manuale su dispositivo fisico Android | 3h | 3h |
| Bug fixing post-test + regressioni | 8h | 7h |
| Ottimizzazioni performance (lazy load, immagini, Firestore indexes) | 4h | 3h |
| **Totale Fase 6** | **27h** | **22.5h** |

---

### Fase 7 — Deploy e Release

| Task | Solo | Con Claude |
|------|------|-----------|
| Firebase deploy production (functions, rules, indexes, hosting) | 2h | 2h |
| Configurazione Stripe production (chiavi live, webhook URL prod) | 2h | 2h |
| Build iOS + certificati + App Store Connect | 5h | 5h |
| Build Android + firma APK + Google Play Console | 4h | 4h |
| **Totale Fase 7** | **13h** | **13h** |

> Le ore di deploy non si riducono con Claude: sono operazioni manuali su portali esterni (Apple, Google, Stripe Dashboard).

---

## Riepilogo Ore e Costi

| Fase | Ore (Solo) | Ore (Con Claude) | Costo (Con Claude) |
|------|-----------|-----------------|-------------------|
| 1 — Setup & Architettura | 18h | 12.5h | €312 |
| 2 — Autenticazione | 14h | 10h | €250 |
| 3 — Cloud Functions Backend | 33h | 23h | €575 |
| 4 — Area Insegnante | 85h | 60.5h | €1.512 |
| 5 — Area Genitore | 30h | 20.5h | €512 |
| 6 — Testing & Rifinitura | 27h | 22.5h | €562 |
| 7 — Deploy & Release | 13h | 13h | €325 |
| **TOTALE** | **220h** | **162h** | **€4.050** |

---

## Pianificazione Weekend

Ogni weekend = 8h (4h sabato + 4h domenica).  
Totale ore: **166h → 21 weekend** (arrotondato per sicurezza).

| Weekend | Date (lun inizio) | Fase | Obiettivo |
|---------|------------------|------|-----------|
| 1 | 05 lug 2026 | F1 | Scaffold, Firebase 3 ambienti, routing, guards |
| 2 | 12 lug 2026 | F1-F2 | Store NgRx, modelli, Auth screens |
| 3 | 19 lug 2026 | F2 | Cloud Function setRole, onboarding push |
| 4 | 26 lug 2026 | F3 | Stripe Payment Intent + Subscription |
| 5 | 02 ago 2026 | F3 | Stripe Webhook + Customer Portal |
| 6 | 09 ago 2026 | F3 | Sistema notifiche FCM + template engine |
| 7 | 16 ago 2026 | F3 | Cron jobs + Firestore Security Rules |
| 8 | 23 ago 2026 | F4 | Dashboard insegnante + Registro Presenze |
| 9 | 30 ago 2026 | F4 | Gestione Iscrizioni + Studenti lista |
| 10 | 06 set 2026 | F4 | Profilo Studente completo (orario, note, storico) |
| 11 | 13 set 2026 | F4 | Calendario (vista + CRUD eventi) |
| 12 | 20 set 2026 | F4 | Avvisi/Bacheca + Gestione Pagamenti |
| 13 | 27 set 2026 | F4 | Comunicazioni insegnante + Chat real-time |
| 14 | 04 ott 2026 | F4 | Impostazioni (template notifiche, chiusure, profilo) |
| 15 | 11 ott 2026 | F5 | Dashboard genitore + Iscrizione figlio |
| 16 | 18 ott 2026 | F5 | Calendario figlio + Pagamenti Stripe mobile |
| 17 | 25 ott 2026 | F5 | Avvisi genitore + Comunicazioni genitore |
| 18 | 01 nov 2026 | F6 | Unit test Store + Cloud Functions |
| 19 | 08 nov 2026 | F6 | Test su dispositivo iOS + Android + bug fixing |
| 20 | 15 nov 2026 | F6 | Performance, regressioni, rifinitura UI |
| 21 | 22 nov 2026 | F7 | Deploy production, App Store, Google Play |

**Fine stimata: weekend del 22-23 novembre 2026**  
**Durata totale: ~5 mesi** (luglio → novembre 2026)

---

## Buffer e Rischi

| Rischio | Probabilità | Impatto | Mitigazione |
|---------|------------|---------|-------------|
| Revisione Apple / Google Play (attesa approvazione) | Alta | +1-2 settimane | Sottomettere con anticipo, preparare materiali store in F8 |
| Cambio API Stripe o Firebase | Bassa | +4-8h | Versioni fissate in package.json |
| Richieste di modifica in corso d'opera | Media | +8-16h | Raccogliere feedback a ogni fine fase |
| Problemi certificati iOS (provisioning) | Media | +2-4h | Configurare Xcode Cloud il prima possibile |

**Buffer consigliato: +2 weekend (16h) → €400**

---

## Totale con Buffer

| Voce | Importo |
|------|---------|
| Sviluppo (162h × €25) | €4.050 |
| Buffer rischi (16h × €25) | €400 |
| **Totale stimato** | **€4.450** |

---

## Costi Esterni — Analisi Dettagliata

I costi qui sotto sono **separati dal preventivo di sviluppo** e ricadono su chi gestisce il servizio (te). Sono suddivisi in una tantum, ricorrenti fissi e variabili.

---

### Firebase (Google Cloud — Piano Blaze)

Il piano Blaze è pay-as-you-go ma include un **free tier generoso** che copre quasi interamente un doposcuola di piccole-medie dimensioni.

#### Pricing Firebase (EU, 2026)

| Servizio | Free tier | Oltre il free tier |
|----------|-----------|-------------------|
| Firestore — letture | 50.000/giorno | $0,06 per 100.000 |
| Firestore — scritture | 20.000/giorno | $0,18 per 100.000 |
| Firestore — storage | 1 GB | $0,18/GB/mese |
| Cloud Functions — invocazioni | 2.000.000/mese | $0,40/milione |
| Cloud Functions — compute | 400.000 GB-sec/mese | $0,0000025/GB-sec |
| Firebase Auth | Gratuito sempre | — |
| FCM (push notifications) | Gratuito sempre | — |
| Firebase Hosting | 10 GB trasferimento/mese | $0,15/GB |
| Crashlytics / Analytics | Gratuito sempre | — |

#### Stima per numero di studenti

| Scenario | Studenti | Utenti totali | Letture/mese | Scritture/mese | Costo Firebase/mese |
|----------|----------|--------------|-------------|----------------|---------------------|
| Avvio | 20 | ~45 | ~400.000 | ~60.000 | **€0** (nel free tier) |
| Crescita | 50 | ~110 | ~900.000 | ~140.000 | **€0–2** |
| Maturo | 100 | ~220 | ~1.800.000 | ~270.000 | **€3–6** |
| Grande | 250 | ~550 | ~4.500.000 | ~650.000 | **€12–18** |

> Le stime di lettura includono: dashboard insegnante, calendario studenti, registro presenze, avvisi, notifiche. Le scritture includono: presenze giornaliere, messaggi chat, pagamenti, eventi.

**Nota importante:** servono un account di fatturazione Google Cloud (carta di credito registrata) anche per restare nel free tier, perché Cloud Functions non è disponibile sul piano Spark gratuito.

**Costo stimato a regime (50 studenti): €0–2/mese**

---

### Stripe

Stripe non ha costi fissi mensili. Addebita solo sulle transazioni effettivamente processate.

#### Tariffe Stripe (EU, carte europee, 2026)

| Tipo transazione | Tariffa |
|-----------------|---------|
| Carta EU standard (Visa, Mastercard) | 1,5% + €0,25 |
| Carta non-EU (es. UK, USA) | 2,5% + €0,25 |
| Stripe Billing (abbonamenti ricorrenti) | +0,5% sul volume abbonamento |
| Stripe Customer Portal | Gratuito |
| Bonifico bancario (SEPA) | 0,2% (min €0,20, max €5,00) |

**Alternativa SEPA Debit:** per ridurre i costi puoi abilitare i pagamenti tramite addebito SEPA diretto (0,2%, massimo €5). Più lento da configurare e richiede il mandato firmato dal genitore, ma può dimezzare i costi su importi superiori a €100.

---

### Apple Developer Program

| Voce | Importo | Frequenza |
|------|---------|-----------|
| Apple Developer Program (account individuale) | **€99/anno** | Annuale |
| Apple Developer Program (account organizzazione) | **€299/anno** | Annuale — richiede DUNS number |
| TestFlight (beta testing) | Incluso | — |
| Xcode Cloud CI/CD — prime 25h compute/mese | Incluso | — |
| Xcode Cloud CI/CD — oltre 25h | $14,99/mese (100h) | Solo se usi CI/CD intensivo |

> Per uso personale/freelance basta l'account individuale a €99/anno. L'account organizzazione serve solo se pubblichi sotto il nome di una società.

**Costo stimato: €99/anno**

---

### Google Play Store

| Voce | Importo | Frequenza |
|------|---------|-----------|
| Registrazione Google Play Developer | **€25** | Una tantum |
| Commissione Google su acquisti in-app | 15% (primo €1M/anno) | Solo su acquisti in-app — non applicabile qui |

> L'app non usa acquisti in-app Google (i pagamenti passano tutti da Stripe), quindi la commissione Google del 15-30% non si applica.

**Costo: €25 una tantum**

---

### Fatturazione Elettronica (opzionale — Fase 2)

Se decidi di emettere fatture elettroniche italiane (SDI) in automatico. Vedi [Pagamenti](./06-pagamenti.md) per il dettaglio.

| Servizio | Piano base | Note |
|----------|-----------|------|
| Fatture in Cloud (Teamsystem) | €6,99/mese | Fino a 20 doc/mese; API disponibile |
| Aruba Fatturazione | €1,99/mese + €0,05/fattura | Molto economico per volumi bassi |
| Fiscozen | €199/anno (tutto incluso) | Include anche consulenza fiscale |

**Costo stimato se implementato: €2–7/mese**

---

### Dominio e Certificati (opzionale)

Firebase fornisce domini gratuiti (`*.web.app`, `*.firebaseapp.com`) sufficienti per le Cloud Functions. Se vuoi un dominio personalizzato (es. `app.ilmiodoposcuola.it`):

| Voce | Importo | Frequenza |
|------|---------|-----------|
| Dominio `.it` | €5–15/anno | Annuale |
| SSL/TLS | Gratuito (Firebase / Let's Encrypt) | — |

---

### Riepilogo Costi Esterni

#### Una tantum (da sostenere prima del lancio)

| Voce | Importo |
|------|---------|
| Google Play Store | €25 |
| Dominio (opzionale) | €10 |
| **Totale una tantum** | **€35** |

#### Ricorrenti fissi (annuali)

| Voce | Importo/anno |
|------|-------------|
| Apple Developer Program | €99 |
| Dominio (opzionale) | €10 |
| **Totale fissi annuali** | **€109/anno** → ~**€9/mese** |

#### Variabili (mensili, dipendono dal volume)

| Servizio | Costo |
|----------|-------|
| Firebase | €0–18/mese (vedi tabella sizing sopra) |
| Stripe | % sul transato — detratta automaticamente da ogni bonifico |
| Fatturazione elettronica (se implementata) | €2–7/mese |

---

### Costo Totale di Gestione — Primo Anno

| Voce | Importo |
|------|---------|
| Sviluppo (con buffer) | €4.450 |
| Google Play Store (una tantum) | €25 |
| Apple Developer Program (1 anno) | €99 |
| Dominio (opzionale, 1 anno) | €10 |
| Firebase (12 mesi) | €0–216 |
| **Totale primo anno** | **~€4.584–4.800** |

> Stripe non è incluso: le commissioni sono trattenute direttamente sul transato e non costituiscono una spesa separata dal servizio.
