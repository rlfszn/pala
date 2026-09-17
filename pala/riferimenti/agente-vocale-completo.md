# Agente vocale completo — tutto quello che si può configurare su ElevenLabs

Riferimento per la skill `/pala:agente-vocale`. Ordine: prima le cose che servono sempre, poi quelle che si aggiungono quando il cliente cresce.

## 1. Le fondamenta (sempre)

| Cosa | Dove | Valore consigliato per una PMI italiana |
|---|---|---|
| Modello del cervello | `agent.prompt.llm` | `claude-sonnet-4-5`, temperatura `0.2`. Più freddo = meno inventa |
| Lingua | `agent.language` | `it` |
| Voce | `tts.voice_id`, `tts.model_id` | `eleven_v3_conversational`, voce italiana naturale, `stability 0.5`, `speed 1.0` |
| Primo messaggio | `agent.first_message` | Nome dell'attività + cosa può fare. Mai "sono un assistente AI generico" |
| Prompt | `agent.prompt.prompt` | Personalità, compito, cosa NON fare, come chiudere |
| Durata massima | `conversation.max_duration_seconds` | 420 s, con messaggio di chiusura gentile |
| Ascolto | `asr.quality: high`, `provider: scribe_realtime` | Tenere alto: sul telefono l'audio è peggiore |
| Turni | `turn.turn_timeout` 7 s, `turn.mode: turn`, `turn_model: turn_v3` | Chi chiama al telefono fa pause lunghe |
| Riempitivi | `turn.soft_timeout_config` | "Un attimo…" dopo 3 s, mentre il tool lavora. Senza, sembra caduta la linea |
| Fuso orario | `agent.prompt.timezone` | `Europe/Rome`, altrimenti sbaglia gli appuntamenti |
| Dizionario di pronuncia | `tts.pronunciation_dictionary_locators` | Per il nome dell'attività, se si legge male |

## 2. Gli strumenti di sistema (gratis, si attivano con un flag)

| Strumento | A cosa serve | Quando accenderlo |
|---|---|---|
| `end_call` | Chiude la chiamata quando ha finito | Sempre |
| `skip_turn` | Sta zitto e aspetta senza riempire il silenzio | Sempre |
| `language_detection` | Passa alla lingua di chi chiama | Zone turistiche, aziende con clienti stranieri |
| `transfer_to_number` | Passa la chiamata a una persona vera | Sempre, se c'è un numero dove passarla. Campi: `transfer_number`, `client_message`, `agent_message`, `reason` |
| `transfer_to_agent` | Passa a un altro agente specializzato | Quando i compiti sono tanti: accoglienza → prenotazioni, assistenza, amministrazione |
| `voicemail_detection` | Capisce se ha risposto una segreteria | Solo per le chiamate in uscita |
| `play_keypad_touch_tone` | Preme i tasti nei centralini | Solo per chiamate in uscita verso aziende |
| `update_state` | Si segna cose dette durante la chiamata senza chiamare niente fuori | Utile per raccogliere dati passo passo |

## 3. Gli strumenti che fanno il lavoro vero (webhook)

Sono le chiamate ai sistemi dell'azienda. Su n8n si costruiscono in mezz'ora l'uno.

| Strumento | Cosa fa | Note |
|---|---|---|
| `verifica_disponibilita` | Legge il calendario e propone due orari | Cal.com o Google Calendar |
| `prenota_appuntamento` | Fissa davvero l'appuntamento | Deve rispondere con conferma e ora |
| `cerca_cliente` | Cerca il numero in arrivo nell'anagrafica | Personalizza subito: "bentornato" |
| `apri_richiesta` | Apre una scheda: guasto, preventivo, reclamo | Finisce nel CRM o in un foglio |
| `manda_whatsapp` | Manda conferma o link dopo la chiamata | Chiude il cerchio anche se la linea cade |
| `stato_ordine` | Legge un ordine o una pratica | Solo se l'azienda ha già un gestionale |

Regole: risposte dello strumento in italiano e brevi; timeout 20 s; se lo strumento fallisce l'agente deve dire "ti faccio richiamare" e aprire una richiesta, mai inventare.

## 4. Sapere le cose dell'azienda (knowledge base + RAG)

- Fonti: file (PDF, Word, Markdown, HTML), URL del sito, testo incollato. Massimo 20 MB a file.
- Documenti brevi (sotto ~300.000 caratteri): l'agente li tiene interi nel prompt.
- Oltre: si accende il RAG e l'agente pesca solo i pezzi utili (`rag.enabled`, `max_retrieved_rag_chunks_count`).
- Per una PMI bastano 4 documenti: orari e indirizzo, listino e prezzi indicativi, domande frequenti, cosa l'agente NON deve promettere.

## 5. Personalizzare ogni chiamata

- **Variabili dinamiche** `{{ nome_cliente }}`, `{{ nome_attivita }}`: si mettono nel prompt e nei tool.
- **Webhook di inizio conversazione**: quando entra una chiamata, ElevenLabs chiede al tuo server chi sta chiamando. Tu rispondi con nome, storico, ultimo appuntamento. L'agente saluta per nome. Funziona su telefono, non sul widget del sito.
- **Overrides**: cambiano prompt, voce o lingua per singola conversazione. Utile per un agente unico che serve più clienti.

## 6. Dopo la chiamata (la parte che fa rinnovare l'abbonamento)

| Cosa | Dove | Perché conta |
|---|---|---|
| Criteri di riuscita | `platform_settings.evaluation.criteria` | Definisci "riuscita" = appuntamento fissato. Il cliente vede la percentuale |
| Raccolta dati | `platform_settings.data_collection` | Estrae dalla chiamata: motivo, servizio richiesto, obiezione, urgenza |
| Sentimento | `sentiment_analysis.enabled` | Segnala i clienti arrabbiati |
| Webhook di fine chiamata | Impostazioni workspace | Manda tutto a n8n: riassunto, esito, dati, trascrizione, costo |
| Report al titolare | n8n | Email o WhatsApp ogni sera: chiamate risposte, appuntamenti presi, chi richiamare |

Il report serale è la cosa che rende visibile il valore: senza, dopo due mesi il titolare pensa di pagare per niente.

## 7. Il telefono

- **Twilio**: numero comprato su Twilio, poi in ElevenLabs servono numero, SID e token (meglio una API key `SK…`). Funziona in entrata e in uscita.
- **Numero esistente dell'azienda**: si può verificare come chiamante per le sole chiamate in uscita, oppure deviare le chiamate del numero vero verso quello nuovo quando nessuno risponde.
- **SIP trunking**: per aziende con centralino proprio.
- Consiglio operativo: non sostituire mai il numero principale al primo giorno. Si parte con la deviazione su "occupato o nessuna risposta": zero rischi e il titolare lo accetta subito.

## 8. Sicurezza e privacy (obbligatorio in Italia)

- `privacy.record_voice: false` se non serve l'audio; `retention_days` breve.
- `guardrails`: attivare i filtri su contenuti e prompt injection quando l'agente è pubblico.
- `call_limits`: tetto giornaliero per non bruciare credito.
- L'agente deve dire che è un assistente automatico se glielo chiedono, e il cliente deve poter parlare con una persona.
- Informativa privacy dell'azienda aggiornata: le chiamate sono trattate da un fornitore esterno.

## 9. Prima di consegnare

- **Test automatici** (`platform_settings.testing`): conversazioni tipo che devono finire bene. Si rilanciano a ogni modifica del prompt.
- **Prova a voce** da telefono vero, non dal computer: l'audio del telefono è un'altra cosa.
- **Tre prove cattive**: chi parla veloce, chi interrompe, chi chiede una cosa fuori tema. L'agente deve reggere tutte e tre.
- **Esperimenti A/B** per confrontare due versioni del prompt con traffico vero.
