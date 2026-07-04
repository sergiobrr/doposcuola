---
title: Riassunto
nav_order: 1
parent: Analisi
---

# App Doposcuola — Riassunto del progetto

← [Indice](./README.md)

**Cos'è:** App mobile iOS + Android per gestire un servizio di doposcuola privato.

---

## Stack

- **Frontend:** Ionic 8 + Angular 20, Capacitor
- **Backend:** Firebase (Firestore, Auth, Cloud Functions, FCM)
- **Pagamenti:** Stripe
- **Stato:** Angular Signals + NgRx SignalStore

---

## Tre ruoli utente

### Insegnante
Creato manualmente da Firebase. Accesso completo:
- Dashboard operativa giornaliera
- Registro presenze quotidiano
- Gestione iscrizioni (approva/rifiuta)
- Anagrafica studenti con orari, note private, storico
- Calendario con CRUD eventi (interrogazioni, compiti, uscite)
- Bacheca avvisi broadcast
- Panoramica pagamenti + export CSV
- Chat con i genitori (solo lui apre i thread)
- Impostazioni: template notifiche, chiusure, profilo

### Genitore
Si auto-registra:
- Iscrive i figli, vede stato iscrizione
- Paga via Stripe (Payment Sheet nativo)
- Calendario del figlio + può aggiungere eventi
- Segnala assenze (form rapido, non una chat)
- Risponde ai thread aperti dall'insegnante
- Bacheca avvisi in lettura
- Può concedere/revocare allo studente il permesso di creare eventi

### Studente
Creato dall'insegnante o dal genitore:
- Calendario settimanale in sola lettura
- Lista prossimi eventi
- Bacheca avvisi in lettura
- Può creare eventi solo se autorizzato da insegnante o genitore

---

## Funzionalità trasversali

### Notifiche push (FCM)
Template configurabili dall'insegnante con variabili `{{nomeStudente}}`, `{{data}}`, ecc. Trigger automatici via Cloud Functions: evento domani, pagamento in scadenza, iscrizione approvata, nuova assenza segnalata, nuovo avviso, ecc.

### Pagamenti Stripe
- Settimanale/mensile → Subscription con rinnovo automatico
- Rimborsi gestiti dalla Stripe Dashboard (non nell'app)
- Customer Portal per il genitore (cambia carta, scarica ricevute)
- Webhook → Cloud Function → aggiorna Firestore

### Chat
Unidirezionale nell'iniziativa: solo l'insegnante apre thread, il genitore risponde. Real-time via Firestore listener. Lo studente non ha accesso.

---

## Database (Firestore)

Collezioni principali: `users`, `studenti`, `iscrizioni`, `eventi`, `presenze`, `assenze`, `avvisi`, `chiusure`, `threads/messages`, `pagamenti`, `notifiche-template`, `config/globale`.

---

## Piano di lavoro

**8 fasi, 21 weekend, 8h/weekend** (4h sabato + 4h domenica)

| Fase | Contenuto | Ore | Costo |
|------|-----------|-----|-------|
| 1 | Setup, Firebase, routing, store | 12.5h | €312 |
| 2 | Autenticazione + onboarding push | 10h | €250 |
| 3 | Cloud Functions + Stripe + notifiche + Security Rules | 23h | €575 |
| 4 | Area insegnante completa | 58h | €1.450 |
| 5 | Area genitore | 20h | €500 |
| 6 | Area studente | 6.5h | €162 |
| 7 | Test + bug fixing + performance | 22.5h | €562 |
| 8 | Deploy App Store + Google Play | 13h | €325 |
| **Totale** | | **165.5h** | **€4.137** |

Con buffer rischi: **€4.537**

**Inizio:** weekend del 5 luglio 2026 — **Fine stimata: novembre 2026**

---

## Costi infrastruttura (primo anno)

| Voce | Importo |
|------|---------|
| Sviluppo (con buffer) | €4.537 |
| Google Play (una tantum) | €25 |
| Apple Developer | €99/anno |
| Firebase | €0–216/anno |
| Stripe | % sul transato (detratta automaticamente) |
| **Totale** | **~€4.700–4.900** |
