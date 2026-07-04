---
title: Pagine e Navigazione
nav_order: 4
parent: Analisi
---

# Pagine e Navigazione

← [Indice](./README.md)

---

## Area Autenticazione (`/auth`)

### Login (`/auth/login`)
Pagina di accesso comune a tutti i ruoli. Dopo il login, il guard legge il Custom Claim `role` e reindirizza all'area corretta (`/teacher`, `/parent`, `/student`).
- Form email + password
- Link "Password dimenticata"
- Link "Registrati" (visibile solo se non autenticati — porta alla registrazione genitore)

### Registrazione Genitore (`/auth/register`)
Solo i genitori si auto-registrano. Insegnante e studenti vengono creati manualmente o dal flusso di iscrizione.
- Form: nome, cognome, email, telefono, password
- Accetta privacy policy
- Dopo la registrazione → email di verifica → redirect al login

### Recupero Password (`/auth/forgot-password`)
Form con campo email. Invia il link di reset tramite Firebase Auth.

---

## Area Insegnante (`/teacher`)

### Dashboard (`/teacher/dashboard`)
Vista riassuntiva della giornata e delle situazioni pendenti. Centro operativo principale.
- Contatore iscrizioni in attesa di approvazione
- Contatore pagamenti scaduti o in scadenza (7 giorni)
- Prossimi eventi del giorno/settimana (interrogazioni, compiti)
- Ultimi messaggi non letti nei thread
- Quick-action: crea evento, invia comunicazione, approva iscrizione

### Gestione Iscrizioni (`/teacher/iscrizioni`)
Lista di tutte le iscrizioni con filtri per stato.
- Tab: "In attesa" / "Attive" / "Tutte"
- Card per ogni iscrizione: nome studente, genitore, tipo, stato pagamento
- Azione: Approva / Rifiuta con nota opzionale
- Azione: Apri dettaglio studente

**Dettaglio Iscrizione (`/teacher/iscrizioni/:id`)**
- Dati completi studente e genitore
- Storico pagamenti
- Pulsante per creare comunicazione con il genitore
- Modifica tipo iscrizione / importo

### Gestione Studenti (`/teacher/studenti`)
Elenco completo degli studenti attivi.
- Ricerca per nome/cognome/classe
- Filtro per stato (attivo, sospeso)
- Card: nome, classe, stato, prossimo evento

**Profilo Studente (`/teacher/studenti/:id`)**
- Dati anagrafici
- Orario settimanale con editor (aggiunta/rimozione slot)
- Lista eventi assegnati (con indicazione di chi li ha creati: insegnante / genitore / studente)
- **Note private** — area visibile solo all'insegnante: lista note con timestamp, aggiungi/modifica/elimina
- Storico presenze mensile (grafico o tabella compatta)
- Storico comunicazioni con i genitori
- Stato iscrizione e pagamenti

### Calendario (`/teacher/calendario`)
Vista del calendario condiviso con tutti gli eventi.
- Vista mensile / settimanale / giornaliera
- Filtro per studente o per tipo evento
- Click su data → crea nuovo evento
- Click su evento → dettaglio/modifica

**Crea/Modifica Evento (`/teacher/calendario/evento/:id`)**
- Tipo evento (interrogazione, compito, uscita, altro)
- Titolo, descrizione, materia
- Data e ora
- Assegna a: tutti gli studenti o selezione manuale
- Numero massimo iscrizioni (capacità dell'occorrenza)
- Toggle "Evento ricorrente" — se attivo:
  - Cadenza: settimanale / mensile
  - Data fine (opzionale; se assente le occorrenze vengono generate a 3 mesi)
  - Numero massimo iscrizioni default per tutta la serie
- Salva → notifica automatica ai destinatari (se template attivo)

**Modifica occorrenza singola vs. serie**
Quando l'insegnante apre un evento che fa parte di una serie, viene chiesto:
- "Modifica solo questa occorrenza" → aggiorna il documento evento, imposta `isModificato = true`
- "Modifica tutta la serie" → aggiorna `serieEventi` e rigenera le occorrenze future non modificate

### Registro Presenze (`/teacher/presenze`)
Operazione quotidiana principale: chi è presente oggi.
- Vista default sul giorno corrente con lista di tutti gli studenti attivi
- Chip per ogni studente: Presente / Assente / Ritardo / Uscita anticipata
- Badge sugli studenti per cui il genitore ha già segnalato assenza (segnalazione pre-compilata)
- Navigazione a ritroso per consultare o correggere giorni precedenti
- Riepilogo mensile presenze per studente (tabella export CSV)

### Gestione Avvisi / Bacheca (`/teacher/avvisi`)
Comunicazioni broadcast verso tutti o un sottogruppo di genitori/studenti.
- Lista avvisi pubblicati con indicatore "letti da X/Y destinatari"
- Toggle "Fissa in evidenza" per avvisi importanti
- FAB: crea nuovo avviso

**Crea/Modifica Avviso (`/teacher/avvisi/:id`)**
- Titolo e testo (editor semplice)
- Destinatari: tutti / solo genitori / solo studenti / selezione manuale per studente
- Pubblica → notifica push ai destinatari (template `nuovo_avviso`)

### Gestione Pagamenti (`/teacher/pagamenti`)
Panoramica finanziaria.
- Riepilogo: incassato nel mese, in scadenza, scaduti
- Lista pagamenti con filtri (stato, periodo, studente)
- Esporta CSV
- Click su pagamento → dettaglio Stripe

### Comunicazioni (`/teacher/comunicazioni`)
Lista di tutti i thread aperti. Vedi [Sistema Comunicazioni](./07-comunicazioni.md).
- Tab: "Aperti" / "Archiviati"
- Card thread: nome genitore/studente, ultimo messaggio, badge non letti
- FAB: crea nuovo thread (seleziona genitore, aggiungi titolo)

**Thread Chat (`/teacher/comunicazioni/:threadId`)**
- Messaggi in ordine cronologico (stile chat)
- Input testo + invio
- Upload allegato (opzionale)
- Pulsante archivia thread

### Impostazioni (`/teacher/impostazioni`)
Configurazioni dell'applicazione.

**Chiusure e Variazioni Orario (`/teacher/impostazioni/chiusure`)**
- Lista chiusure programmate future
- Crea chiusura: titolo, tipo, data inizio/fine, nota opzionale
- Salva → notifica push ai destinatari se la chiusura è entro 7 giorni
- Le chiusure appaiono sul calendario come giorni non disponibili

**Template Notifiche (`/teacher/impostazioni/notifiche`)**
Lista e configurazione dei template. Vedi [Notifiche](./05-notifiche.md).
- Toggle attiva/disattiva per ogni template
- Modifica testo titolo e corpo (con variabili `{{nome}}`, `{{data}}`, ecc.)
- Seleziona canali (push, in-app)
- Seleziona destinatari (genitori, studenti, entrambi)

**Profilo (`/teacher/impostazioni/profilo`)**
- Modifica dati personali
- Cambio password

---

## Area Genitore (`/parent`)

### Dashboard Genitore (`/parent/dashboard`)
Vista rapida della situazione dei propri figli.
- Card per ogni figlio: nome, prossimo evento, stato pagamento
- **Pulsante rapido "Segnala assenza oggi"** su ogni card figlio — apre un bottom sheet con campo motivazione opzionale e conferma con un tap. Non apre un thread di chat.
- Badge notifiche non lette
- Accesso rapido a comunicazioni con l'insegnante
- Avvisi in evidenza dalla bacheca (massimo 2, con link "Vedi tutti")

### I Miei Figli (`/parent/figli`)
Lista dei figli associati al genitore.
- Card figlio: nome, classe, stato iscrizione
- Pulsante "Iscrivi nuovo figlio"

**Iscrizione Nuovo Figlio (`/parent/figli/iscrizione`)**
- Form dati studente: nome, cognome, data nascita, classe, scuola
- Seleziona tipo iscrizione (settimanale, mensile)
- Conferma → richiesta inviata all'insegnante

**Calendario Figlio (`/parent/figli/:studentId/calendario`)**
- Vista mensile/settimanale
- Evidenzia interrogazioni e compiti in classe
- **FAB "Aggiungi evento"** — il genitore può sempre creare eventi per il proprio figlio
- Click evento → dettaglio (titolo, materia, descrizione, chi lo ha creato)

### Pagamenti (`/parent/pagamenti`)
- Lista pagamenti: data, importo, stato, descrizione
- Pulsante "Paga ora" per quote in attesa → redirect a pagina di checkout
- Accesso al Stripe Customer Portal (gestione carta)

**Checkout (`/parent/pagamenti/checkout/:iscrizioneId`)**
- Pagina statica con descrizione del servizio (testo, orari, modalità)
- Riepilogo importo e tipo iscrizione
- Pulsante "Prosegui" → apre Stripe Payment Sheet nativo

### Bacheca Avvisi (`/parent/avvisi`)
- Lista avvisi pubblicati dall'insegnante, ordinati per data (fissati in cima)
- Badge con numero avvisi non letti
- Click avviso → testo completo, segna automaticamente come letto

### Comunicazioni (`/parent/comunicazioni`)
- Lista thread aperti dall'insegnante verso il genitore
- Solo lettura dei thread (il genitore non può crearne di nuovi)
- Il genitore può rispondere nei thread esistenti
- Badge con messaggi non letti

**Thread Chat (`/parent/comunicazioni/:threadId`)**
- Identica alla vista insegnante ma senza possibilità di creare thread
- Input risposta abilitato

### Profilo Genitore (`/parent/profilo`)
- Dati personali (nome, email, telefono)
- Gestione consenso notifiche push
- Cambio password

---

## Componenti Condivisi Notevoli

| Componente | Utilizzo |
|------------|----------|
| `NotificheBadgeComponent` | Badge nell'header con contatore push non lette |
| `EventoCardComponent` | Card riusabile per visualizzare un evento |
| `CalendarioWeekComponent` | Vista settimanale condivisa tra insegnante e genitore |
| `ChatMessageComponent` | Bolla messaggio nella chat |
| `PagamentoStatusBadgeComponent` | Badge colorato stato pagamento |
| `StudenteCardComponent` | Card riassuntiva studente |
