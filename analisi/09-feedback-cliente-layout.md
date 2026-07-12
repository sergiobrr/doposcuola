---
title: Feedback Cliente — Layout Alternativo
nav_order: 10
parent: Analisi
---

# Feedback Cliente — Analisi Layout "AfterSkull"

← [Indice](./README.md)

## Contesto

La cliente ha risposto al link/documentazione condivisa con:

> "Secondo me è molto più snello di così quello che vorrei... ma poi dimmi tu... sto lavorando su un layout per spiegarmi meglio."

Ha allegato 4 foto di schizzi a mano (`immagini_cliente/0.jpeg` → `3.jpeg`), numerati "1", "2", "3" (le prime due immagini sono due versioni — bozza e refinement — della stessa schermata "1"). Ha annunciato che sta preparando un layout più completo da inviare a breve: **questo report va quindi trattato come lettura preliminare, non come specifica definitiva.**

Nome prodotto che emerge dagli schizzi: **"AfterSkull"** (naming diverso da "Doposcuola" usato finora).

---

## Cosa mostrano gli schizzi

### Foto 0 e 1 — Schermata "1": Home / Catalogo Servizi
Bozza e versione più ordinata della stessa schermata.

- **Apertura app**: logo "AfterSkull" + elemento grafico (icona/faccina), mini testo descrittivo: *"Spazio pomeridiano di aiuto assistito per i ragazzi dai 10 ai 16 anni"*
- **Claim**: "Scopri e scegli il tuo servizio" / "Guarda e scegli il tuo servizio"
- **Login**: "Oppure accedi o registrati" — quindi si può *sfogliare* il catalogo anche senza account, il login serve solo per prenotare
- **Tre servizi verticali**, ciascuno come card/riquadro con proprio orario:
  - **Homeworks** — Lun-Ven 14:30-16:30
  - **Extra / Lezioni Private** — Lun-Mer-Ven 17:00-18:00
  - **Laboratori** — Mar-Gio 17:00-18:30
- Per ogni servizio: verifica disponibilità → form (nome/cognome, data nascita, scuola, dati genitore, campo "extra") → tipo booking (giornaliero €20 o mensile intero €300) → iscrivi/paga

### Foto 2 — Schermata "3": Riepilogo Homeworks + Prenotazione + Pagamento
- **Riepilogo servizio**: "Homeworks", Lun-Ven 14:30-16:30, prezzo mese (es. "intero 300") o al singolo incontro
- **Metodo di pagamento**: esplicitamente "quello che vuoi" → PayPal, Carta, ecc. — poi "Concludi" → "Scarica ricevuta"
- **Prenota**: calendario con giorni specifici cliccabili (Lun/Mer/Ven, numerati 13-15-17, 21-23-25, 28...) — quindi prenotazione **per singola data**, non solo abbonamento forfettario
- Nota a margine: *"la cosa più efficace vedi tu"* — lascia libertà di scelta UX su come mostrare il calendario
- **Form iscrizione**: nome/cognome, data nascita, scuola, dati genitore, extra, prezzo, totale
- Menzione di **"interrogazione"**, **"aiuto nel metodo di studio"**, **"BES"** (Bisogni Educativi Speciali) come possibili "tipo" di richiesta/motivazione — segnale che vuole distinguere il bisogno specifico del ragazzo

### Foto 3 — Schermata Laboratori
- Versione "molto semplice": Mar-Gio 17:00-18:30
- Piccolo testo descrittivo del laboratorio
- Iscrizione con lab/costo di default, tipo booking
- Nome/cognome, data nascita → Concludi/Paga
- **Menu a tendina** (probabilmente header/hamburger) con: **Contatti**, **Il mio account** (profilo utente), **I pagamenti**, **Le mie prenotazioni**

---

## Confronto con l'analisi attuale (`analisi/00-08`)

| Elemento | Specifica attuale | Richiesta cliente (schizzi) |
|---|---|---|
| Ruoli | Insegnante (admin) + Genitore, con aree separate e guard | Solo utente genitore che sfoglia/prenota; **nessuna interfaccia insegnante disegnata** |
| Iscrizione | Genitore invia richiesta → **stato "in attesa" → approvazione insegnante** | Booking diretto, sembra confermato subito dal pagamento (come prenotare un volo/hotel) |
| Servizi | Eventi/serie ricorrenti generici (interrogazioni, compiti, uscite), gestiti da calendario CRUD insegnante | 3 categorie fisse e ben definite: **Homeworks, Lezioni Private/Extra, Laboratori**, ciascuna con orario proprio |
| Calendario | Vista mensile/settimanale complessa, con occorrenze, serie, override singola occorrenza | Calendario semplice con giorni cliccabili disponibili/non disponibili |
| Presenze | Registro giornaliero insegnante (presente/assente/ritardo) | Non presente negli schizzi |
| Comunicazioni | Chat thread unidirezionale insegnante↔genitore | Non presente negli schizzi — solo "Contatti" nel menu (probabilmente form/telefono/email statici) |
| Notifiche push | Sistema template configurabile con trigger automatici | Non presente negli schizzi |
| Dashboard insegnante | Centro operativo con contatori, quick action | Assente |
| Pagamenti | Solo Stripe (Payment Sheet nativo + Subscription) | PayPal + Carta esplicitamente citati come opzioni ("quello che vuoi") |
| Note private studente, storico, orario editor | Presenti | Assenti |
| Account genitore | Profilo con gestione consenso notifiche | "Il mio account" generico nel menu |
| Sezione richieste speciali | Assente | Accenno a "BES" / "aiuto nel metodo di studio" come possibile tipo di richiesta |

**In sintesi:** la cliente immagina un'app molto più vicina a una **vetrina di servizi con booking e pagamento** (modello simile a un'app di prenotazione corsi/lezioni), mentre la documentazione attuale descrive un **gestionale scolastico completo** (CRM con approvazioni, registro presenze, calendario ricorrente avanzato, chat, motore notifiche).

---

## Si riparte da zero?

**Sul piano tecnico: no.** Ionic + Angular + Firebase + Stripe restano scelte valide e anzi più che sufficienti — probabilmente sovradimensionate rispetto al nuovo scope più leggero. Non c'è nulla negli schizzi che richieda di cambiare stack.

**Sul piano della specifica funzionale/prodotto: sì, in gran parte.** I documenti `02-architettura.md`, `03-pagine.md`, `04-modello-dati.md` descrivono un prodotto diverso da quello che la cliente sta disegnando. Andrebbero riscritti su questa base più snella, non solo corretti. Di conseguenza anche `08-preventivo.md` andrebbe rifatto: lo scope negli schizzi è nettamente più piccolo (niente registro presenze, niente calendario ricorrente con occorrenze/serie, niente chat real-time, niente motore notifiche configurabile, niente dashboard insegnante) — verosimilmente **meno della metà delle ore** stimate finora (le fasi 3, 4 e parte della 6 nel preventivo attuale si ridurrebbero drasticamente).

`01-stack-tecnologico.md` resta quasi tutto valido, salvo aggiungere PayPal come metodo di pagamento accanto a Stripe (da valutare se serve davvero o se Stripe da solo con Apple/Google Pay copre già l'esigenza — vedi domande sotto).

---

## Domande aperte da porre alla cliente prima di aggiornare l'analisi

1. **Serve comunque un pannello di gestione per lei** (vedere chi si è iscritto, bloccare/sbloccare disponibilità, incassi) o pensa di gestire tutto "a mano" fuori dall'app (es. guardando i pagamenti su Stripe/PayPal e i dati via email)?
2. Le iscrizioni/prenotazioni **richiedono un'approvazione manuale** o si confermano automaticamente al pagamento, come sembra dagli schizzi?
3. I tre servizi (Homeworks, Extra/Lezioni Private, Laboratori) hanno **posti limitati con calendario reale** (con giorni/slot che si esauriscono) o sono acquisto libero di un pacchetto/abbonamento?
4. Le "prenotazioni" viste nella foto 3 sono **per singola data** (es. prenoto il 13, il 15, il 17...) oppure è un abbonamento mensile e il calendario serve solo a vedere quando si tiene il servizio?
5. Serve gestione **multi-figlio** per uno stesso genitore, come nel modello attuale, o ogni prenotazione è pensata per un solo bambino alla volta?
6. **PayPal** è un requisito reale o un'idea abbozzata? Se serve solo "carta", Stripe da solo (con Apple Pay/Google Pay) potrebbe bastare e semplificare l'integrazione.
7. Cosa deve contenere la voce **"Contatti"** nel menu — form di contatto, telefono/email statici, o anche qui una chat?
8. Le note tipo *"BES"* e *"aiuto nel metodo di studio"* sono un vero requisito (es. campo del form per segnalare bisogni specifici del ragazzo) o solo appunti mentali della cliente durante lo schizzo?

---

## Prossimi passi consigliati

1. **Aspettare il layout più definito** che la cliente ha detto di voler mandare — questi schizzi sono ancora bozze di lavoro, non una specifica.
2. Nel frattempo **non modificare né iniziare a scrivere codice**: l'app attuale (se già scaffoldata) resta ferma finché lo scope non è chiaro.
3. Una volta ricevuto il nuovo layout e le risposte alle domande sopra, **riscrivere** `02-architettura.md`, `03-pagine.md`, `04-modello-dati.md` sul nuovo scope, e **ricalcolare** `08-preventivo.md` (probabile riduzione significativa di ore/costo, quindi da comunicare positivamente alla cliente).
