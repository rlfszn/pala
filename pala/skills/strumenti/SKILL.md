---
description: Prepara gli strumenti che servono per lavorare: account ElevenLabs, n8n con la chiave API, Twilio quando serve il telefono. Usala al primo avvio, prima della demo, o quando manca una chiave.
---

# PALA · Strumenti

Prepari **tu** il terreno. L'utente deve solo cliccare dove gli dici e incollare quello che gli chiedi.

Regole: una cosa alla volta, conferma prima di procedere, mai parole tecniche senza mostrarle. Se si blocca, fatti incollare quello che vede e risolvi tu.

## Dove si salvano le chiavi

Tutte in `~/.pala/chiavi.env`, un valore per riga. Crea la cartella e il file la prima volta e spiega in una riga: "queste chiavi restano sul tuo computer, non le vede nessun altro".

```
ELEVENLABS_API_KEY=
N8N_URL=
N8N_API_KEY=
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
```

Non chiedere mai tutte le chiavi insieme. Ognuna si chiede **quando serve**, non prima:
- ElevenLabs → prima della demo
- n8n → quando l'agente deve prendere appuntamenti
- Twilio → quando si collega il telefono vero

## 1. ElevenLabs (serve per la voce)

1. Vai su elevenlabs.io e registrati. Il piano gratuito basta per costruire e provare.
2. In alto a destra clicca sulla tua immagine, poi **API Keys**, poi **Create API Key**. Dai un nome, per esempio "pala", e copia la chiave: si vede una volta sola.
3. Incollala qui: la salvo io nel file.

Poi verifica subito che funzioni chiamando l'API degli agenti. Se la chiave è sbagliata dillo e fai rifare il passo, senza spiegazioni lunghe.

**Cosa si paga**: costruire e provare è gratis. Si paga quando l'agente parla davvero, a minuti di conversazione. Dillo chiaro: quel costo lo mette il cliente nel canone mensile, non l'utente.

## 2. n8n (serve perché l'agente faccia cose)

n8n è il posto dove vivono i collegamenti: calendario, WhatsApp, email, il riepilogo serale al titolare. Spiegalo così: "è la centralina che collega l'agente agli strumenti dell'azienda".

Due strade, e vanno dette entrambe con i costi veri:

**Strada A, n8n Cloud.** Registrazione su n8n.io, prova gratuita di 14 giorni. **Attenzione: durante la prova gratuita la chiave API non è disponibile.** Serve il piano Starter, circa 24€ al mese. Con la chiave, i workflow li costruisco io al posto suo.

**Strada B, n8n sul proprio server.** Versione Community, gratuita per sempre, e la chiave API c'è. Si installa su un server da circa 5€ al mese. Più economica, ma richiede mezz'ora di installazione guidata.

Come consigliare:
- Se sta ancora provando e non ha clienti: **strada B**, e lo guidi nell'installazione.
- Se ha già il primo cliente che paga: **strada A**, perché 24€ al mese su un canone da 300 non si sentono e si risparmia tempo.
- Se non vuole spendere niente adesso: si può fare la demo senza n8n. L'agente parla, ma non prenota. Per il primo incontro basta.

Quando ha n8n:
1. Dentro n8n: **Settings**, poi **n8n API**, poi **Create an API key**. Nome "pala", scadenza lunga.
2. Copia la chiave e l'indirizzo del suo n8n (quello che vede nella barra del browser, senza niente dopo il dominio).
3. Incolla qui tutti e due: li salvo io.

Verifica la chiave con una chiamata all'elenco dei workflow. Se risponde, dì solo: "collegato, da qui in poi i flussi li costruisco io".

**Cosa cambia per l'utente**: con la chiave, l'utente non deve imparare n8n. Io creo i workflow via API, lui li vede comparire nel suo pannello e li accende.

## 3. Twilio (serve solo per il telefono vero)

Da fare **solo** quando il cliente ha detto sì e si passa alla consegna.

1. Registrazione su twilio.com. Il credito di prova basta per le prime chiamate.
2. Compra un numero italiano: **Phone Numbers**, poi **Buy a number**, filtro Italia, con capacità Voce. Costa pochi euro al mese.
3. Prendi **Account SID** e **Auth Token** dalla pagina iniziale, meglio ancora crea una API Key dedicata.
4. Incolla qui: li salvo io e collego il numero all'agente.

Ricorda la regola: il numero dell'azienda non si tocca mai. Si devia solo su occupato o nessuna risposta.

## Riepilogo da mostrare quando serve

Quando l'utente chiede "quanto mi costa tutto", rispondi con questa tabella e niente altro:

| Cosa | Quando si paga | Quanto |
|---|---|---|
| Claude | Sempre | circa 20€ al mese |
| ElevenLabs | Solo quando l'agente parla con i clienti | a minuti, lo paga il cliente nel canone |
| n8n | Dal primo cliente | 0€ sul proprio server, 24€ al mese su Cloud |
| Numero di telefono | Alla consegna | pochi euro al mese, lo paga il cliente |

Il primo cliente ripaga tutto. Dal secondo guadagna.
