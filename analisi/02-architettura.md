---
title: Architettura
nav_order: 3
parent: Analisi
---

# Architettura dell'Applicazione

← [Indice](./README.md)

## Ruoli e Permessi

| Ruolo | Come si crea | Permessi |
|-------|--------------|----------|
| `teacher` | Manualmente da admin Firebase | Accesso completo a tutto |
| `parent` | Auto-registrazione | Iscrizioni, pagamenti, comunicazioni (risposta), calendario figli, creazione eventi per i propri figli, **segnalazione assenze**, bacheca avvisi (lettura) |

I ruoli sono gestiti tramite **Firebase Custom Claims** impostati da una Cloud Function. Il token JWT contiene il claim `role`, letto dalla app al login per determinare il routing.

---

## Struttura delle Aree

```
App
├── /auth
│   ├── login
│   ├── register          (solo genitori)
│   └── forgot-password
│
├── /teacher              (guard: role === 'teacher')
│   ├── dashboard
│   ├── presenze
│   ├── iscrizioni
│   ├── calendario
│   ├── studenti
│   ├── avvisi
│   ├── pagamenti
│   ├── comunicazioni
│   └── impostazioni
│       ├── notifiche-template
│       ├── chiusure
│       └── profilo
│
├── /parent               (guard: role === 'parent')
│   ├── dashboard
│   ├── figli
│   │   ├── iscrizione
│   │   └── [studentId]/calendario
│   ├── pagamenti
│   │   └── checkout/:iscrizioneId   (descrizione servizio + prosegui a Stripe)
│   ├── avvisi
│   ├── comunicazioni
│   └── profilo
│
```

---

## Gestione Stato — Architettura a Livelli

### Livello 1 — Signal locale (componente)
Stato UI transitorio, non condiviso.
```typescript
// Esempio: apertura/chiusura modal
protected isModalOpen = signal(false);
protected selectedDate = signal<Date | null>(null);
```

### Livello 2 — Injectable Service con Signal (feature-scoped)
Stato condiviso tra componenti della stessa feature, non persistito nel router.
```typescript
// Esempio: CalendarioService
@Injectable({ providedIn: 'root' })
export class CalendarioService {
  private _eventi = signal<Evento[]>([]);
  eventi = this._eventi.asReadonly();
  settimanaCorrente = signal(startOfWeek(new Date()));
}
```

### Livello 3 — NgRx SignalStore (app-wide)
Stato globale accessibile ovunque nell'app.
```typescript
// Esempio: AuthStore, NotificheStore
export const AuthStore = signalStore(
  { providedIn: 'root' },
  withState<AuthState>({ utente: null, ruolo: null, loading: false }),
  withMethods(...)
);
```

### Schema dei Store globali

```
AuthStore          → utente corrente, ruolo, token
NotificheStore     → notifiche non lette, badge count
ComunicazioniStore → thread aperti, messaggi non letti
```

---

## Flusso di Autenticazione

```
1. App avvia → AuthStore.init() → controlla Firebase Auth state
2. Se autenticato → legge Custom Claim `role`
3. Router Guard controlla il ruolo → redirect all'area corretta
4. Se non autenticato → redirect a /auth/login
```

---

## Flusso Iscrizione Genitore

```
1. Genitore si registra (/auth/register)
2. Cloud Function imposta claim role=parent
3. Genitore crea profilo figlio (/parent/figli/iscrizione)
4. Insegnante riceve notifica nuova iscrizione in attesa
5. Insegnante approva/rifiuta (/teacher/iscrizioni)
6. Se approvata → Stripe Payment Intent creato
7. Genitore apre /parent/pagamenti → preme "Paga ora"
8. Pagina descrizione servizio (/parent/pagamenti/checkout/:iscrizioneId)
9. Genitore preme "Prosegui" → Stripe Payment Sheet nativo
10. Webhook Stripe → Cloud Function aggiorna stato su Firestore
11. Studente diventa attivo (stato `attivo` su Firestore)
```

---

## Considerazioni Evolutive

### Limite Posti e Lista d'Attesa
Il documento `config/globale` su Firestore contiene `limitePostiMassimo` e `listaAttesaAbilitata`. Quando un genitore invia una richiesta di iscrizione, una Cloud Function verifica il numero di studenti attivi prima di accettarla. Se il limite è raggiunto e la lista d'attesa è abilitata, la richiesta viene messa in coda con stato `in_attesa_posto` invece di `in_attesa`.

### Multi-Insegnante (Fase 2)
Il modello attuale supporta un solo insegnante (`role === 'teacher'` senza distinzione). Per scalare a più insegnanti senza riscrivere tutto:
- Il Custom Claim resta `role: 'teacher'`
- `config/globale.insegnanteIds` tiene la lista degli uid autorizzati
- Le Firestore Security Rules vengono aggiornate per verificare `insegnanteIds.hasAny([request.auth.uid])` invece del solo claim
- Si aggiunge un concetto di "studenti assegnati" per dividere il carico tra insegnanti, se necessario

---

## Struttura Cartelle Progetto (Ionic/Angular)

```
src/
├── app/
│   ├── core/
│   │   ├── guards/          (auth.guard, role.guard)
│   │   ├── interceptors/    (auth-token.interceptor)
│   │   ├── services/        (firebase.service, auth.service)
│   │   └── stores/          (auth.store, notifiche.store)
│   │
│   ├── shared/
│   │   ├── components/      (componenti riusabili)
│   │   ├── pipes/
│   │   └── models/          (interfacce TypeScript)
│   │
│   ├── features/
│   │   ├── teacher/
│   │   ├── parent/
│   │   └── student/
│   │
│   └── auth/
│
├── environments/
│   ├── environment.ts       (dev)
│   ├── environment.staging.ts
│   └── environment.prod.ts
│
└── firebase/
    └── functions/           (Cloud Functions TypeScript)
```
