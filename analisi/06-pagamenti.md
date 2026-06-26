---
title: Pagamenti Stripe
nav_order: 7
parent: Analisi
---

# Pagamenti Stripe

← [Indice](./README.md)

## Panoramica

I pagamenti sono gestiti interamente tramite **Stripe**, senza che i dati della carta transitino per i nostri server. Il flusso utilizza:
- **Payment Intents** per i pagamenti una tantum (es. iscrizione annuale)
- **Subscriptions** per quote mensili o trimestrali ricorrenti
- **Customer Portal** per permettere ai genitori di cambiare il metodo di pagamento

---

## Tipi di Iscrizione e Pagamento

| Tipo | Meccanismo Stripe | Note |
|------|-------------------|------|
| Mensile | Subscription (monthly) | Rinnovo automatico |
| Trimestrale | Subscription (quarterly) | Rinnovo automatico |
| Annuale | Payment Intent (one-time) | O Subscription annual |

---

## Flusso Completo

### 1. Creazione Customer Stripe

Alla prima approvazione di un'iscrizione, una Cloud Function crea il Customer Stripe:

```typescript
// Cloud Function: onIscrizioneApprovata
const customer = await stripe.customers.create({
  email: genitore.email,
  name: `${genitore.nome} ${genitore.cognome}`,
  metadata: { parentId: genitore.uid, studenteId: studente.id }
});

await updateDoc(iscrizioneRef, {
  stripeCustomerId: customer.id
});
```

### 2. Creazione Payment Intent / Subscription

```typescript
// Per pagamento una tantum (annuale):
const paymentIntent = await stripe.paymentIntents.create({
  amount: importoInCentesimi,
  currency: 'eur',
  customer: stripeCustomerId,
  metadata: { iscrizioneId, studenteId }
});

// Per abbonamento (mensile/trimestrale):
const subscription = await stripe.subscriptions.create({
  customer: stripeCustomerId,
  items: [{ price: priceId }],  // Price configurato in Stripe Dashboard
  payment_behavior: 'default_incomplete',
  expand: ['latest_invoice.payment_intent'],
  metadata: { iscrizioneId, studenteId }
});
```

### 3. Pagamento nel Client (Ionic/Capacitor)

```typescript
// Usa @capacitor-community/stripe per la Payment Sheet nativa
import { Stripe } from '@capacitor-community/stripe';

await Stripe.createPaymentSheet({
  paymentIntentClientSecret: clientSecret,
  merchantDisplayName: 'Doposcuola',
  customerId: stripeCustomerId,
  customerEphemeralKeySecret: ephemeralKey,
});

const result = await Stripe.presentPaymentSheet();
if (result.paymentResult === PaymentSheetEventsEnum.Completed) {
  // UI ottimistica: mostra "pagamento in elaborazione"
  // Il webhook confermerà e aggiornerà lo stato su Firestore
}
```

### 4. Webhook Stripe → Cloud Function

```typescript
// Cloud Function: stripeWebhook
app.post('/stripe-webhook', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(req.body, sig, WEBHOOK_SECRET);

  switch (event.type) {
    case 'payment_intent.succeeded':
      await aggiornaStatoPagamento(event.data.object, 'succeeded');
      await inviaNotiticaRicevuta(event.data.object.metadata.iscrizioneId);
      break;

    case 'payment_intent.payment_failed':
      await aggiornaStatoPagamento(event.data.object, 'failed');
      break;

    case 'invoice.payment_failed':   // abbonamento non rinnovato
      await gestisciPagamentoAbbonamentoFallito(event.data.object);
      break;

    case 'customer.subscription.deleted':
      await disattivaIscrizione(event.data.object.metadata.iscrizioneId);
      break;
  }

  res.json({ received: true });
});
```

---

## Customer Portal (Gestione Carta)

Il genitore può accedere al Stripe Customer Portal per:
- Cambiare metodo di pagamento
- Scaricare le fatture
- Annullare (se consentito) l'abbonamento

```typescript
// Cloud Function: createPortalSession
const session = await stripe.billingPortal.sessions.create({
  customer: stripeCustomerId,
  return_url: 'doposcuola://parent/pagamenti'  // deep link app
});
// Apri session.url nel browser in-app
await Browser.open({ url: session.url });
```

---

## Struttura Dati Pagamenti su Firestore

Ogni evento Stripe crea/aggiorna un documento in `pagamenti/{pagamentoId}`. Vedi [Modello Dati](./04-modello-dati.md).

---

## Gestione Rimborsi

I rimborsi vengono gestiti dall'insegnante tramite la **Stripe Dashboard** (non nell'app, per semplicità). Il webhook `charge.refunded` aggiorna lo stato su Firestore.

---

## Prezzi e Prodotti Stripe

Configurati staticamente nella Stripe Dashboard per ambiente:

| Prodotto | Price ID (dev) | Frequenza |
|----------|----------------|-----------|
| Quota mensile | `price_dev_mensile` | monthly |
| Quota trimestrale | `price_dev_trimestrale` | ogni 3 mesi |
| Quota annuale | `price_dev_annuale` | one-time |

Gli ID vengono letti da `environment.ts` per ogni ambiente.

---

## Fatturazione Italiana

Le ricevute automatiche di Stripe **non sono fatture fiscali italiane**. Per la compliance fiscale italiana ci sono due strade:

### Opzione A — Export dati per il commercialista (consigliata per avvio)
- Pagina `/teacher/pagamenti` include un export CSV mensile con: data, importo, genitore, studente, descrizione
- Nessuna integrazione aggiuntiva, zero costi
- Il commercialista gestisce la fatturazione manualmente

### Opzione B — Integrazione fatturazione elettronica (fase 2)
Integrazioni compatibili con il Sistema di Interscambio (SDI):

| Servizio | SDK/API | Note |
|----------|---------|-------|
| **Fatture in Cloud** (Teamsystem) | REST API | Molto diffuso, piano gratuito limitato |
| **Aruba Fatturazione** | REST API | Economico |
| **Fiscozen** | REST API | Orientato ai freelance |

Il flusso sarebbe: webhook Stripe `payment_intent.succeeded` → Cloud Function → API fatturazione → emette fattura → la allega al documento `pagamenti/{id}` come URL PDF.

> **Detrazione spese istruzione**: i genitori potrebbero aver bisogno delle ricevute per la detrazione fiscale del 19% (art. 15 TUIR). Anche in Opzione A, Stripe genera ricevute scaricabili dal Customer Portal che possono essere usate a questo scopo.

---

## Sicurezza

- Le chiavi Stripe **secret** sono solo nelle Cloud Functions (variabili d'ambiente Firebase)
- Il client usa solo la **publishable key** e i `clientSecret` temporanei
- I webhook sono verificati con la firma HMAC di Stripe
- Nessun dato di carta viene mai toccato dalla nostra applicazione
