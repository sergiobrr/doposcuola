---
title: Sistema Notifiche
nav_order: 6
parent: Analisi
---

# Sistema Notifiche

← [Indice](./README.md)

## Canali Supportati

| Canale | Descrizione |
|--------|-------------|
| **Push** | Notifica su dispositivo tramite FCM (visibile anche con app chiusa) |
| **In-App** | Banner/alert mostrato all'interno dell'app quando è aperta |

Le due modalità sono indipendenti e configurabili per template.

---

## Template di Notifica

L'insegnante configura i template da `/teacher/impostazioni/notifiche`. Ogni template è un documento Firestore in `notifiche-template/{templateId}`.

### Struttura Template

```typescript
interface NotificaTemplate {
  id: string;
  nome: string;                         // etichetta per l'insegnante
  evento: TipoEvento;                   // trigger automatico
  titolo: string;                       // supporta variabili {{...}}
  corpo: string;                        // supporta variabili {{...}}
  canali: ('push' | 'inApp')[];
  destinatari: 'parent'[];               // chi riceve
  attivo: boolean;
}
```

### Variabili Disponibili nei Template

| Variabile | Valore |
|-----------|--------|
| `{{nomeStudente}}` | Nome e cognome dello studente |
| `{{nomeGenitore}}` | Nome del genitore |
| `{{tipoEvento}}` | Tipo evento (es. "Interrogazione") |
| `{{materiaEvento}}` | Materia dell'evento |
| `{{dataEvento}}` | Data formattata (es. "lunedì 12 gennaio") |
| `{{importo}}` | Importo pagamento (es. "€ 150,00") |
| `{{dataScadenza}}` | Data scadenza pagamento |
| `{{giorniMancanti}}` | Giorni mancanti all'evento/scadenza |
| `{{creatoreEvento}}` | Chi ha creato l'evento (es. "il genitore Mario Rossi", "lo studente") |
| `{{titoloAvviso}}` | Titolo dell'avviso in bacheca |
| `{{dataChiusura}}` | Data della chiusura/variazione orario |
| `{{titoloChiusura}}` | Titolo della chiusura (es. "Ponte del 2 giugno") |

---

## Tipi di Evento Trigger

### Automatici (gestiti da Cloud Functions)

| Evento | Quando si attiva | Destinatari tipici |
|--------|------------------|--------------------|
| `evento_domani` | Ogni giorno alle 18:00, se c'è un evento il giorno dopo | parent |
| `evento_tra_3_giorni` | Ogni giorno alle 18:00, se c'è un evento tra 3 giorni | parent |
| `pagamento_scaduto` | Il giorno della scadenza del pagamento | parent |
| `pagamento_in_scadenza` | 7 giorni prima della scadenza | parent |
| `pagamento_ricevuto` | Webhook Stripe: pagamento confermato | parent |
| `iscrizione_approvata` | Insegnante approva l'iscrizione | parent |
| `iscrizione_rifiutata` | Insegnante rifiuta l'iscrizione | parent |
| `nuovo_messaggio` | Insegnante invia un messaggio in un thread | parent |
| `evento_creato_da_genitore` | Genitore crea un evento per il figlio | teacher |
| `assenza_segnalata` | Genitore segnala assenza del figlio | teacher |
| `chiusura_programmata` | Insegnante aggiunge una chiusura nei prossimi 7 giorni | parent |
| `nuovo_avviso` | Insegnante pubblica un avviso in bacheca | parent |

### Manuali (inviate direttamente dall'insegnante)

L'insegnante può inviare una notifica manuale selezionando un template e uno o più destinatari dalla lista studenti/genitori. Utile per comunicazioni urgenti o avvisi generali.

---

## Flusso Tecnico Notifica Push

```
1. Trigger (automatico o manuale)
         │
         ▼
2. Cloud Function `sendNotification`
   - Legge il template attivo per quell'evento
   - Risolve le variabili con i dati Firestore
   - Raccoglie i fcmTokens degli utenti destinatari
         │
         ▼
3. Firebase Admin SDK → FCM
   - Invia a ogni token con titolo/corpo risolti
         │
         ▼
4. Dispositivo riceve la push
   - Capacitor Push Notifications plugin la intercetta
   - Se app aperta → mostra banner in-app
   - Se app chiusa → notifica di sistema nativa
         │
         ▼
5. Scrive log in `notifiche-log/{logId}`
   - templateId, destinatari, timestamp, stato (sent/failed)
```

---

## Gestione Token FCM

I token FCM cambiano quando l'utente reinstalla l'app o svuota i dati. Ogni utente può avere più dispositivi.

```typescript
// Al login e ogni volta che il token viene rinnovato:
PushNotifications.addListener('registration', async (token) => {
  await updateDoc(doc(db, 'users', userId), {
    fcmTokens: arrayUnion(token.value)
  });
});
```

La Cloud Function rimuove automaticamente i token non validi (errore `messaging/registration-token-not-registered`).

---

## Configurazione UI — Pagina Template

La pagina `/teacher/impostazioni/notifiche` mostra una lista di card, una per template.

```
┌─────────────────────────────────────────────────┐
│ 📅 Evento domani                      [●] Attivo │
│                                                  │
│ Titolo: "Domani: {{tipoEvento}} di {{materia}}" │
│ Corpo:  "Ricorda a {{nomeStudente}} ..."         │
│                                                  │
│ Canali: [✓] Push  [✓] In-App                    │
│ A:      [✓] Genitore                             │
│                                    [Modifica]    │
└─────────────────────────────────────────────────┘
```

---

## Consenso Notifiche Push (GDPR / App Store)

Al primo avvio dell'app, dopo il login:
1. Mostra modal esplicativo "Vuoi ricevere notifiche per eventi e pagamenti?"
2. Se accetta → `PushNotifications.requestPermissions()` → salva token
3. Se rifiuta → salva preferenza, non chiede di nuovo automaticamente
4. Il genitore può modificare il consenso dal proprio profilo

```typescript
// Richiesta permesso (solo se non già concesso/negato)
const result = await PushNotifications.requestPermissions();
if (result.receive === 'granted') {
  await PushNotifications.register();
}
```
