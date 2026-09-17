---
description: Costruisce l'agente vocale per un'azienda cliente, dal primo prototipo alla consegna. Usala quando l'utente deve fare la demo, costruire l'agente, collegare il telefono o consegnare il sistema al cliente.
---

# PALA · Agente vocale

Costruisci **tu** l'agente al posto dell'utente. Lui non deve toccare la configurazione: deve capire cosa sta succedendo, provare la voce e mostrarla al cliente.

Il riferimento completo di tutto ciò che si può configurare è in `${CLAUDE_PLUGIN_ROOT}/riferimenti/agente-vocale-completo.md`. Leggilo prima di costruire, e segui l'ordine: prima le fondamenta, poi il resto.

Valgono le regole di conversazione della skill `inizia`: una domanda alla volta, niente termini tecnici senza mostrarli, ogni passo finisce con una cosa fatta, conferma prima di procedere.

## Prima di iniziare: cosa serve

Servono due account, entrambi gratuiti per provare. Se mancano, li fai aprire ora, un passo alla volta:

1. **ElevenLabs** (elevenlabs.io): registrazione, poi Settings → API Keys → Create. La chiave si salva in `~/.pala/chiavi.env` come `ELEVENLABS_API_KEY=...`. Spiega che il piano gratuito basta per la demo e che si paga solo quando l'agente parla davvero.
2. **n8n** serve solo quando l'agente deve fare qualcosa (prenotare, scrivere in agenda). Per la demo non serve: rimandalo.

Se l'utente si blocca sulla registrazione, guidalo schermata per schermata. Non dare per scontato niente.

## Passo 1 — Capire l'azienda (5 domande, una alla volta)

1. Come si chiama l'attività e cosa fa, in una riga.
2. Chi chiama e perché? (esempi: prendere un appuntamento, chiedere un preventivo, sapere gli orari)
3. Cosa succede oggi quando nessuno risponde?
4. Cosa l'agente non deve mai fare o promettere? (prezzi fissi, urgenze, consigli medici)
5. C'è un numero dove passare la chiamata se serve una persona?

Salva tutto in `~/.pala/clienti/<nome-azienda>.json`.

## Passo 2 — La demo, in dieci minuti

Costruisci l'agente via API con la chiave dell'utente. Configurazione minima ma già solida:

- lingua italiana, voce italiana naturale, modello `eleven_v3_conversational`
- `claude-sonnet-4-5` a temperatura 0.2, fuso `Europe/Rome`
- primo messaggio con il nome dell'attività
- prompt con: chi sei, cosa fai, cosa non fai mai, come chiudi
- strumenti `end_call` e `skip_turn` accesi
- riempitivo "un attimo…" dopo 3 secondi
- durata massima 7 minuti

Poi dagli il link del widget di prova e digli: **chiamalo adesso, parla come se fossi un cliente**. Aspetta che l'abbia fatto.

Se qualcosa suona male, sistemalo tu: voce troppo veloce, risposte troppo lunghe, non capisce una parola. Una modifica alla volta, e fagli riprovare.

Questa è la demo da mostrare al cliente. Non serve altro per il primo incontro.

## Passo 3 — Farlo lavorare (dopo il sì del cliente)

Ora l'agente deve fare cose, non solo parlare. Nell'ordine:

1. **Prendere appuntamenti**: strumento webhook verso il calendario del cliente. Servono due strumenti, uno che legge la disponibilità e uno che prenota. Costruiscili su n8n, e guida l'utente nel collegare il calendario.
2. **Sapere le cose dell'azienda**: quattro documenti nella knowledge base. Orari e indirizzo, listino indicativo, domande frequenti, cosa non promettere.
3. **Passare a una persona**: `transfer_to_number` verso il cellulare indicato al passo 1.
4. **Raccontare la giornata**: raccolta dati e criterio di riuscita, più il webhook di fine chiamata verso n8n che manda al titolare il riepilogo serale.

Spiega ogni pezzo in una riga, in italiano, prima di costruirlo. Mai elencare tutto insieme.

## Passo 4 — Il telefono

Regola d'oro, dilla al cliente: **non si cambia il numero dell'azienda.** Si compra un numero nuovo su Twilio, si collega all'agente, e il numero vero devia su quello solo quando è occupato o non risponde nessuno.

Così il titolare non rischia niente: se l'agente sbaglia, lui se ne accorge e la sua linea resta com'era.

Guida l'utente: numero su Twilio, poi in ElevenLabs servono numero, SID e token. Poi una chiamata di prova vera, da telefono, non dal computer.

## Passo 5 — Le prove che evitano le figuracce

Prima di consegnare, fai provare tre chiamate difficili:
- uno che parla veloce e interrompe
- uno che chiede una cosa fuori tema
- uno che chiede il prezzo esatto

L'agente deve reggere tutte e tre senza inventare. Se inventa, stringi il prompt e rifai la prova.

Poi crea i test automatici, così ogni volta che si cambia il prompt si ricontrolla da solo.

## Passo 6 — La consegna

Prepara per il cliente, in una pagina sola:
- cosa fa l'agente e cosa non fa
- il numero da chiamare per provarlo
- come arriva il riepilogo serale
- chi chiamare se qualcosa non va (l'utente)

E ricorda all'utente la cosa che conta: il sistema resta acceso finché il cliente paga il canone. È il suo lavoro tenerlo vivo, guardare i riepiloghi una volta a settimana e migliorare una cosa al mese.

## Cosa non fare mai

- Non consegnare un agente che non è stato provato al telefono vero.
- Non promettere al cliente che l'agente sostituisce una persona: risponde quando la persona non c'è.
- Non registrare l'audio delle chiamate senza dirlo e senza informativa.
- Non lasciare l'agente senza un modo per passare a un umano.
