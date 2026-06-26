---
title: Home
nav_order: 1
---

# Doposcuola App

Documentazione completa del progetto per la realizzazione di un'applicazione mobile dedicata alla gestione di un servizio di doposcuola.

## Sezioni

| Documento | Descrizione |
|-----------|-------------|
| [Stack Tecnologico](./analisi/01-stack-tecnologico) | Framework, librerie, servizi cloud e motivazioni delle scelte |
| [Architettura](./analisi/02-architettura) | Struttura dell'applicazione, ruoli utente, flussi principali |
| [Pagine e Navigazione](./analisi/03-pagine) | Dettaglio di ogni pagina per ruolo utente |
| [Modello Dati](./analisi/04-modello-dati) | Struttura del database, collezioni e relazioni |
| [Sistema Notifiche](./analisi/05-notifiche) | Template, trigger, canali di invio e configurazione |
| [Pagamenti Stripe](./analisi/06-pagamenti) | Flusso di pagamento, webhook e gestione abbonamenti |
| [Sistema Comunicazioni](./analisi/07-comunicazioni) | Chat unidirezionale, thread, moderazione |
| [Preventivo](./analisi/08-preventivo) | Stima ore, costi e pianificazione weekend |

## Ruoli Utente

| Ruolo | Accesso |
|-------|---------|
| **Insegnante** | Gestione completa: iscrizioni, presenze, pagamenti, orari, eventi, notifiche, comunicazioni |
| **Genitore** | Iscrizione figli, pagamenti, bacheca avvisi, comunicazioni (risposta), segnalazione assenze |
| **Studente** | Solo lettura: orari, eventi, bacheca avvisi; creazione eventi se autorizzato |
