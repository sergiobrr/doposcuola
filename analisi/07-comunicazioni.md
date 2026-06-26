---
title: Comunicazioni
nav_order: 8
parent: Analisi
---

# Sistema Comunicazioni

← [Indice](./README.md)

## Canali di Comunicazione — Quadro Generale

| Canale | Chi inizia | Chi risponde | Scope |
|--------|-----------|--------------|-------|
| **Thread chat** | Solo insegnante | Genitore (nel thread) | 1-a-1, contestuale |
| **Avvisi / Bacheca** | Solo insegnante | Nessuno (broadcast) | 1-a-molti, informativo |
| **Segnalazione assenza** | Solo genitore | Nessuno (notifica a insegnante) | Azione puntuale, non conversazione |

> La **segnalazione assenza** è l'unica azione che il genitore può iniziare autonomamente, ma non è una conversazione: è un form strutturato che genera una notifica all'insegnante senza aprire un thread.

---

## Principio di Funzionamento

Il sistema di chat è **unidirezionale nell'iniziativa**:

- L'**insegnante** è l'unico che può **aprire** un nuovo thread di conversazione
- Il **genitore** può solo **rispondere** nei thread già aperti verso di lui
- Lo **studente** non ha accesso alle comunicazioni (solo notifiche passive)

Questo evita che l'insegnante sia sommerso da messaggi spontanei e mantiene la comunicazione strutturata e tracciabile.

---

## Struttura Dati

```
threads/{threadId}
  ├── createdByTeacherId: string
  ├── participantIds: [teacherId, parentId]
  ├── studenteId?: string          (se il thread riguarda un figlio)
  ├── titolo: string
  ├── ultimoMessaggio: string
  ├── ultimoMessaggioAt: Timestamp
  ├── archiviato: boolean
  └── messages/{messageId}
        ├── senderId: string
        ├── senderRole: 'teacher' | 'parent'
        ├── testo: string
        ├── allegatoUrl?: string
        ├── letto: boolean
        └── createdAt: Timestamp
```

Vedi dettaglio completo: [Modello Dati](./04-modello-dati.md).

---

## Flusso: Insegnante apre un Thread

```
1. Insegnante va su /teacher/comunicazioni
2. Preme FAB "Nuovo messaggio"
3. Seleziona genitore (e opzionalmente il figlio a cui si riferisce)
4. Inserisce titolo del thread e primo messaggio
5. Invia → crea documento in `threads/` + primo documento in `messages/`
6. Cloud Function invia notifica push al genitore (template: `nuovo_messaggio`)
```

---

## Flusso: Genitore risponde

```
1. Genitore riceve push → apre /parent/comunicazioni/:threadId
   oppure entra nell'app → vede badge non letti → apre il thread
2. Legge la cronologia messaggi
3. Scrive risposta nel campo testo → invia
4. Il messaggio appare in real-time anche nella vista dell'insegnante (Firestore listener)
5. L'insegnante riceve notifica push (se non ha l'app aperta sul thread)
```

---

## Real-Time con Firestore

```typescript
// Listener messaggi in real-time
const messagesQuery = query(
  collection(db, `threads/${threadId}/messages`),
  orderBy('createdAt', 'asc')
);

onSnapshot(messagesQuery, (snapshot) => {
  const messaggi = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
  this._messaggi.set(messaggi);
  this.segnaComeLetti(messaggi);
});
```

---

## Lettura Messaggi

Quando un utente apre un thread, i messaggi non letti vengono marcati:

```typescript
async segnaComeLetti(messaggi: Message[]) {
  const nonLetti = messaggi.filter(
    m => !m.letto && m.senderId !== this.currentUserId
  );
  const batch = writeBatch(db);
  nonLetti.forEach(m => {
    batch.update(doc(db, `threads/${threadId}/messages/${m.id}`), { letto: true });
  });
  await batch.commit();
}
```

Il badge nell'header si aggiorna automaticamente grazie al listener nel `ComunicazioniStore`.

---

## Badge e Contatori Non Letti

```typescript
// ComunicazioniStore (NgRx SignalStore, app-wide)
export const ComunicazioniStore = signalStore(
  { providedIn: 'root' },
  withState({ threadNonLetti: 0 }),
  withMethods((store) => ({
    inizializzaListener(userId: string) {
      // Conta i thread con almeno un messaggio non letto per l'utente corrente
      // Query su Firestore con collectionGroup('messages')
    }
  }))
);
```

---

## Regole di Accesso

| Azione | Insegnante | Genitore | Studente |
|--------|-----------|----------|----------|
| Crea thread | ✅ | ❌ | ❌ |
| Legge thread | ✅ (tutti) | ✅ (solo i propri) | ❌ |
| Scrive messaggio | ✅ | ✅ (solo thread propri) | ❌ |
| Archivia thread | ✅ | ❌ | ❌ |

Queste regole sono imposte sia nelle Firestore Security Rules che nei guard lato client.

---

## Notifiche per Nuovi Messaggi

Quando arriva un nuovo messaggio in un thread, una Cloud Function:
1. Verifica che il destinatario non sia già attivo sul thread (euristica: ultimo accesso)
2. Se assente → invia push tramite template `nuovo_messaggio`
3. Aggiorna `ultimoMessaggio` e `ultimoMessaggioAt` sul documento thread (per preview)

---

## Archivio Thread

L'insegnante può archiviare un thread concluso. I thread archiviati:
- Non mostrano badge di non letti
- Sono visibili nella tab "Archiviati"
- Rimangono leggibili (storico)
- Il genitore non può più rispondere in un thread archiviato

---

## Allegati (Opzionale — Fase 2)

Upload di immagini o PDF tramite **Firebase Storage**:
- Max 5 MB per allegato
- Solo da parte dell'insegnante nella fase iniziale
- Preview inline nell'interfaccia chat
- URL firmato con scadenza per sicurezza
