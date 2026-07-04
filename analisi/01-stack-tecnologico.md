---
title: Stack Tecnologico
nav_order: 2
parent: Analisi
---

# Stack Tecnologico

← [Indice](./README.md)

## Frontend

### Ionic + Angular
- **Ionic 7** con componenti nativi iOS/Android
- **Angular 21** con standalone components
- **Angular Signals** per la reattività locale ai componenti
- **NgRx SignalStore** per lo stato condiviso tra feature (iscrizioni, calendario, notifiche)
- **Angular Router** con lazy loading per ogni area funzionale

### Gestione Stato — Livelli

```
Livello 1 — Signal locale (componente)
  Esempio: stato del form di iscrizione, toggle UI

Livello 2 — Service con signal (feature-scoped)
  Esempio: lista bambini del genitore corrente, orario settimanale

Livello 3 — NgRx SignalStore (app-wide)
  Esempio: utente autenticato, notifiche non lette, configurazioni globali
```

### Navigazione e Tab
```
/auth          → login, registrazione, recupero password
/teacher/*     → area insegnante (guard: ruolo TEACHER)
/parent/*      → area genitore (guard: ruolo PARENT)
```

---

## Backend — Opzione Raccomandata: Firebase

### Perché Firebase

| Criterio | Firebase | Supabase | Backend custom (NestJS) |
|----------|----------|----------|------------------------|
| Push notifications native | ✅ FCM integrato | ⚠️ via servizi terzi | ⚠️ via FCM/APNs manuale |
| Real-time per la chat | ✅ Firestore real-time | ✅ Postgres realtime | ⚠️ WebSocket manuale |
| Auth multi-ruolo | ✅ Custom Claims | ✅ RLS + roles | ✅ JWT custom |
| Setup iniziale | Rapido | Medio | Lento |
| Hosting functions | ✅ Cloud Functions | ✅ Edge Functions | Serve infrastruttura |
| Costo per piccoli volumi | Gratuito (Spark) | Gratuito (500 MB) | Costo VPS minimo |
| SQL se necessario | ❌ NoSQL | ✅ PostgreSQL | ✅ a scelta |

**Scelta:** Firebase per il prototipo e la fase iniziale. Migrazione a Supabase possibile se emergerà la necessità di query SQL complesse.

### Servizi Firebase utilizzati

- **Firebase Auth** — autenticazione email/password + Custom Claims per i ruoli (`teacher`, `parent`, `student`)
- **Cloud Firestore** — database principale (vedi [Modello Dati](./04-modello-dati.md))
- **Firebase Cloud Messaging (FCM)** — push notification su iOS e Android (vedi [Notifiche](./05-notifiche.md))
- **Cloud Functions** — logica server-side: webhook Stripe, invio notifiche programmate, validazioni
- **Firebase Storage** — upload di allegati nelle comunicazioni (opzionale)

---

## Pagamenti

### Stripe
- **Stripe Payment Intents** per pagamenti una tantum (iscrizione)
- **Stripe Subscriptions** per quote mensili ricorrenti
- **Stripe Customer Portal** per i genitori (gestione metodo di pagamento)
- Webhook via **Cloud Function** per aggiornare lo stato dei pagamenti su Firestore
- SDK: `@capacitor-community/stripe` per la UI nativa su mobile

Vedi dettaglio: [Pagamenti Stripe](./06-pagamenti.md)

---

## Notifiche Push

- **Capacitor Push Notifications plugin** (`@capacitor/push-notifications`) per la ricezione
- **FCM** per l'invio server-side
- **Cloud Functions** per trigger automatici (es. pagamento scaduto, evento imminente)
- Template configurabili dall'insegnante (vedi [Notifiche](./05-notifiche.md))

---

## Comunicazioni In-App

- Chat real-time su **Firestore** (collection `threads` → `messages`)
- Solo l'insegnante può creare un nuovo thread
- I genitori possono rispondere nei thread aperti verso di loro
- Vedi: [Comunicazioni](./07-comunicazioni.md)

---

## Dipendenze principali

```json
{
  "@ionic/angular": "^8.x",
  "@angular/core": "^21.x",
  "@ngrx/signals": "^21.x",
  "@angular/fire": "^19.x",
  "@capacitor/core": "^7.x",
  "@capacitor/push-notifications": "^7.x",
  "@capacitor-community/stripe": "^5.x",
  "firebase": "^11.x"
}
```

---

## Ambienti

```
development   → Firebase project: doposcuola-dev
staging       → Firebase project: doposcuola-staging
production    → Firebase project: doposcuola-prod
```

Ogni ambiente ha la propria configurazione Stripe (chiavi test / live).
