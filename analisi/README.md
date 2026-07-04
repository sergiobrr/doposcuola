---
title: Analisi
nav_order: 2
has_children: true
---

# Doposcuola App — Documentazione di Progetto

Applicazione mobile per la gestione di un servizio di doposcuola, sviluppata con **Ionic + Angular**.

## Indice

| Documento | Descrizione |
|-----------|-------------|
| [Riassunto](./00-riassunto.md) | Panoramica sintetica dell'intero progetto |
| [Stack Tecnologico](./01-stack-tecnologico.md) | Framework, librerie, servizi cloud e motivazioni delle scelte |
| [Architettura](./02-architettura.md) | Struttura dell'applicazione, ruoli utente, flussi principali |
| [Pagine e Navigazione](./03-pagine.md) | Dettaglio di ogni pagina per ruolo utente |
| [Modello Dati](./04-modello-dati.md) | Struttura del database, collezioni e relazioni |
| [Sistema Notifiche](./05-notifiche.md) | Template, trigger, canali di invio e configurazione |
| [Pagamenti Stripe](./06-pagamenti.md) | Flusso di pagamento, webhook e gestione abbonamenti |
| [Sistema Comunicazioni](./07-comunicazioni.md) | Chat unidirezionale, thread, moderazione |
| [Preventivo](./08-preventivo.md) | Stima ore, costi e pianificazione weekend |

## Ruoli Utente

```
Insegnante (admin)
  └── Gestione completa: iscrizioni, pagamenti, orari, eventi, notifiche, comunicazioni

Genitore
  └── Iscrizione figli, pagamenti, comunicazioni (solo risposta), calendario figlio

Studente
  └── Solo lettura: orari personali, eventi imminenti
```

## Avanzamento

- [ ] Definizione stack backend
- [ ] Prototipo struttura dati
- [ ] Mockup pagine principali
- [ ] Implementazione autenticazione
- [ ] Implementazione iscrizioni
- [ ] Integrazione Stripe
- [ ] Sistema notifiche
- [ ] Sistema chat
