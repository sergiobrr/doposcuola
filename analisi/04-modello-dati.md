---
title: Modello Dati
nav_order: 5
parent: Analisi
---

# Modello Dati — Firestore

← [Indice](./README.md)

## Struttura Collezioni

```
users/{userId}
iscrizioni/{iscrizioneId}
studenti/{studenteId}
eventi/{eventoId}
orari/{orarioId}
threads/{threadId}
  └── messages/{messageId}
pagamenti/{pagamentoId}
notifiche-template/{templateId}
notifiche-log/{logId}
```

---

## `users/{userId}`

Documento creato al primo login. `userId` = Firebase Auth UID.

```typescript
interface User {
  uid: string;
  email: string;
  displayName: string;
  role: 'teacher' | 'parent' | 'student';
  telefono?: string;
  fcmTokens: string[];        // token push per ogni dispositivo
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

---

## `studenti/{studenteId}`

```typescript
interface Studente {
  id: string;
  nome: string;
  cognome: string;
  dataNascita: Timestamp;
  classe: string;             // es. "3A"
  scuola: string;
  parentIds: string[];        // uid dei genitori associati
  stato: 'in_attesa' | 'attivo' | 'sospeso' | 'terminato';
  permessoEventi: PermessoEventi | null;
  note?: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

// null = nessun permesso concesso (default)
interface PermessoEventi {
  abilitato: boolean;
  concessoDa: string;           // uid di chi ha concesso il permesso
  concessoDaRuolo: 'teacher' | 'parent';
  concessoAt: Timestamp;
}
```

Il permesso può essere concesso dall'**insegnante** oppure da uno dei **genitori** dello studente. Basta che uno dei due lo conceda. Può essere revocato solo da chi lo ha concesso oppure dall'insegnante (sempre).

---

## `iscrizioni/{iscrizioneId}`

```typescript
interface Iscrizione {
  id: string;
  studenteId: string;
  parentId: string;
  statoIscrizione: 'in_attesa' | 'approvata' | 'rifiutata';
  statoPagamento: 'non_pagato' | 'pagato' | 'scaduto' | 'rimborsato';
  tipoIscrizione: 'mensile' | 'trimestrale' | 'annuale';
  importo: number;            // in centesimi (per Stripe)
  stripeCustomerId?: string;
  stripeSubscriptionId?: string;
  dataInizio: Timestamp;
  dataFine?: Timestamp;
  note?: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

---

## `eventi/{eventoId}`

Interrogazioni, compiti in classe, uscite, ricorrenze. Possono essere creati dall'insegnante, da un genitore per il proprio figlio, o dallo studente stesso se ha il permesso.

```typescript
interface Evento {
  id: string;
  tipo: 'interrogazione' | 'compito_in_classe' | 'uscita' | 'altro';
  titolo: string;
  descrizione?: string;
  data: Timestamp;
  studenteIds: string[];      // a chi è assegnato ([] = tutti gli studenti)
  materia?: string;
  completato: boolean;
  createdBy: string;          // uid di chi ha creato l'evento
  createdByRuolo: 'teacher' | 'parent' | 'student';
  updatedAt: Timestamp;
  createdAt: Timestamp;
}
```

> Un genitore può creare eventi solo per i propri figli (`studenteIds` deve essere sottoinsieme di `studente.parentIds`). Uno studente può creare eventi solo per se stesso e solo se `permessoEventi.abilitato === true`.

---

## `presenze/{presenzaId}`

Registro giornaliero delle presenze, compilato dall'insegnante.

```typescript
interface Presenza {
  id: string;
  data: Timestamp;              // normalizzata a mezzanotte (solo il giorno)
  studenteId: string;
  stato: 'presente' | 'assente' | 'in_ritardo' | 'uscita_anticipata';
  nota?: string;                // es. "arrivato alle 15:30"
  registrataDa: string;         // uid insegnante
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

---

## `assenze/{assenzaId}`

Segnalazioni di assenza inviate dal genitore prima o durante il giorno.

```typescript
interface AssenzaGenitore {
  id: string;
  studenteId: string;
  parentId: string;
  data: Timestamp;              // giorno dell'assenza segnalata
  motivazione?: string;
  letta: boolean;               // true quando l'insegnante ha preso visione
  createdAt: Timestamp;
}
```

> Diverso da `presenze`: l'assenza è la **comunicazione del genitore**, la presenza è il **registro ufficiale dell'insegnante**. I due documenti coesistono e sono collegati tramite `studenteId` + `data`.

---

## `avvisi/{avvisoId}`

Comunicazioni broadcast dell'insegnante verso tutti i genitori/studenti o un sottogruppo.

```typescript
interface Avviso {
  id: string;
  titolo: string;
  testo: string;
  createdByTeacherId: string;
  destinatari: ('parent' | 'student')[];
  studenteIds: string[];        // [] = tutti gli studenti attivi
  lettoDa: string[];            // array di uid che hanno aperto l'avviso
  fissato: boolean;             // avviso in evidenza in cima alla bacheca
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

---

## `chiusure/{chiusuraId}`

Eccezioni al calendario: festività, pause, variazioni orario temporanee.

```typescript
interface Chiusura {
  id: string;
  titolo: string;               // es. "Ponte del 2 giugno"
  tipo: 'festivita' | 'pausa_programmata' | 'straordinaria' | 'variazione_orario';
  dataInizio: Timestamp;
  dataFine: Timestamp;          // uguale a dataInizio per chiusura di un giorno
  nota?: string;
  createdAt: Timestamp;
}
```

---

## `studenti/{studenteId}/note/{noteId}`

Note private dell'insegnante sullo studente. Non visibili a genitori né allo studente.

```typescript
interface NotaPrivata {
  id: string;
  testo: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

---

## `config/globale` (documento singleton)

```typescript
interface ConfigGlobale {
  limitePostiMassimo: number;
  listaAttesaAbilitata: boolean;
  orarioAperturaDefault: string;  // "14:00"
  orarioChiusuraDefault: string;  // "18:00"
  insegnanteIds: string[];        // supporto futuro multi-insegnante
}
```

---

## `orari/{orarioId}`

Slot settimanali ricorrenti del doposcuola.

```typescript
interface Orario {
  id: string;
  studenteId: string;
  giornoSettimana: 0 | 1 | 2 | 3 | 4 | 5 | 6;  // 0=Dom, 1=Lun...
  oraInizio: string;          // "14:30"
  oraFine: string;            // "17:30"
  attivo: boolean;
  validoDal: Timestamp;
  validoAl?: Timestamp;
}
```

---

## `threads/{threadId}` e `messages/{messageId}`

```typescript
interface Thread {
  id: string;
  createdByTeacherId: string;
  participantIds: string[];   // teacher + parent(s)
  studenteId?: string;        // se il thread riguarda uno studente specifico
  titolo: string;
  ultimoMessaggio?: string;
  ultimoMessaggioAt?: Timestamp;
  archiviato: boolean;
  createdAt: Timestamp;
}

interface Message {
  id: string;
  threadId: string;
  senderId: string;
  senderRole: 'teacher' | 'parent';
  testo: string;
  allegatoUrl?: string;
  letto: boolean;
  createdAt: Timestamp;
}
```

---

## `pagamenti/{pagamentoId}`

Log dei pagamenti ricevuti da Stripe via webhook.

```typescript
interface Pagamento {
  id: string;
  iscrizioneId: string;
  studenteId: string;
  parentId: string;
  stripePaymentIntentId: string;
  importo: number;            // centesimi
  valuta: string;             // 'eur'
  stato: 'pending' | 'succeeded' | 'failed' | 'refunded';
  descrizione: string;
  createdAt: Timestamp;
}
```

---

## `notifiche-template/{templateId}`

Vedi [Sistema Notifiche](./05-notifiche.md) per il dettaglio.

```typescript
interface NotificaTemplate {
  id: string;
  nome: string;
  evento: string;             // es. 'pagamento_scaduto', 'evento_domani'
  titolo: string;             // es. "Pagamento in scadenza"
  corpo: string;              // es. "Il pagamento di {{nome}} scade il {{data}}"
  canali: ('push' | 'inApp')[];
  destinatari: ('parent' | 'student')[];
  attivo: boolean;
  createdAt: Timestamp;
}
```

---

## Regole di Sicurezza Firestore (sintesi)

```javascript
// Un genitore può leggere solo i propri studenti
match /studenti/{studenteId} {
  allow read: if request.auth.token.role == 'teacher'
    || resource.data.parentIds.hasAny([request.auth.uid])
    || request.auth.uid == studenteId;   // lo studente legge se stesso
  allow write: if request.auth.token.role == 'teacher';

  // Insegnante o genitore del figlio possono modificare solo il campo permessoEventi
  allow update: if (request.auth.token.role == 'teacher'
    || resource.data.parentIds.hasAny([request.auth.uid]))
    && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['permessoEventi']);
}

// Lettura eventi
match /eventi/{eventoId} {
  allow read: if request.auth.token.role == 'teacher'
    || resource.data.studenteIds.hasAny([request.auth.uid])
    || resource.data.studenteIds.size() == 0;

  // Creazione: teacher sempre; genitore per i propri figli; studente solo se ha permesso
  allow create: if request.auth.token.role == 'teacher'
    || (request.auth.token.role == 'parent'
        && genitoreHaFigli(request.auth.uid, request.resource.data.studenteIds))
    || (request.auth.token.role == 'student'
        && studenteHaPermesso(request.auth.uid)
        && request.resource.data.studenteIds == [request.auth.uid]);

  // Modifica/cancellazione: solo chi ha creato o l'insegnante
  allow update, delete: if request.auth.token.role == 'teacher'
    || resource.data.createdBy == request.auth.uid;
}

// Chat: solo i partecipanti del thread possono leggere
match /threads/{threadId} {
  allow read: if request.auth.uid in resource.data.participantIds;
  allow create: if request.auth.token.role == 'teacher';  // solo teacher apre
  allow update: if request.auth.uid in resource.data.participantIds;
}
```

> Le funzioni `genitoreHaFigli` e `studenteHaPermesso` sono helper Firestore Rules che leggono il documento `studenti` per verificare rispettivamente che i `studenteIds` dell'evento siano figli del genitore, e che lo studente abbia `permessoEventi.abilitato == true`.
