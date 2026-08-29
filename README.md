# Ranji 🤖

Ranji è un bot Discord con AI (Grok/xAI), moderazione, logging, e una particolarità:
**impara il tono con cui parlarti**. Più interagisci con lei/lui, più le sue risposte
si adattano al tuo modo di scrivere — formale o informale, scherzoso o serio, paziente
o diretto.

## ✨ Funzionalità

- **Chat con AI**: menziona `@Ranji` in un messaggio (o rispondi a un suo messaggio) per parlare con lei/lui, powered by [Grok (xAI)](https://x.ai).
- **Tono adattivo**: ogni messaggio che scrivi nel server viene analizzato (senza salvare il testo grezzo) e aggiorna un profilo di tono per te — formalità, cordialità, umorismo, pazienza. Il profilo pesa sulle risposte AI future.
- **Moderazione**: `/kick`, `/ban`, `/timeout`, `/warn`, `/warnings`, `/clear`.
- **Logging**: canale di log configurabile (`/setlogchannel`) per azioni di moderazione, messaggi modificati/eliminati, ingressi/uscite dal server.
- **Persistenza leggera**: tutto salvato in un file JSON locale (`data/db.json`), zero database esterni da gestire.

## 📁 Struttura del progetto

```
ranji-bot/
├── src/
│   ├── ai/                # Client Grok + motore che traduce il tono in prompt
│   ├── commands/
│   │   ├── moderation/    # /kick /ban /timeout /warn /warnings /clear
│   │   └── utility/       # /ping /help /setlogchannel /resettone
│   ├── events/            # ready, messaggi, moderazione automatica dei log, ecc.
│   ├── handlers/          # caricamento dinamico di comandi ed eventi
│   ├── utils/             # database JSON, logger, analizzatore di tono
│   ├── config.js
│   ├── deploy-commands.js # registra i comandi slash su Discord
│   └── index.js           # entry point
├── data/                  # db.json (creato automaticamente, ignorato da git)
├── .env.example
└── package.json
```

## 🚀 Setup

### 1. Prerequisiti
- Node.js ≥ 18.17
- Un'applicazione Discord con bot creata su [discord.com/developers/applications](https://discord.com/developers/applications)
- Una API key Grok da [console.x.ai](https://console.x.ai)

### 2. Configura il bot su Discord
Nel Developer Portal, sezione **Bot**:
- Attiva l'intent privilegiato **Server Members Intent**
- Attiva l'intent privilegiato **Message Content Intent**

Nella sezione **OAuth2 > URL Generator**, seleziona lo scope `bot` e `applications.commands`,
più i permessi che ti servono (almeno: Kick Members, Ban Members, Moderate Members,
Manage Messages, Send Messages, Read Message History). Usa il link generato per invitare
il bot sul tuo server.

### 3. Installazione

```bash
git clone <url-del-tuo-repo>
cd ranji-bot
npm install
cp .env.example .env
```

Apri `.env` e compila:
- `DISCORD_TOKEN` — dal Dev Portal, sezione Bot
- `CLIENT_ID` — dal Dev Portal, sezione General Information
- `XAI_API_KEY` — dalla console xAI
- (opzionale) `GUILD_ID` — ID del tuo server, per registrare i comandi istantaneamente durante lo sviluppo

### 4. Registra i comandi slash

```bash
npm run deploy
```

Da rilanciare ogni volta che aggiungi o modifichi un comando.

### 5. Avvia il bot

```bash
npm start
```

Per lo sviluppo, con riavvio automatico ai cambi di file:

```bash
npm run dev
```

## 🧠 Come funziona l'apprendimento del tono

Non è un modello di machine learning: è deliberatamente semplice e leggibile in
`src/utils/toneAnalyzer.js`. Ogni messaggio viene analizzato con euristiche
(parole gentili/scortesi, formalità, emoji, maiuscolo, punteggiatura) e produce un
piccolo "delta" su 4 assi. Questo delta si somma al profilo esistente dell'utente
con una media mobile esponenziale, così il tono cambia gradualmente nel tempo invece
di saltare da un messaggio all'altro. Il profilo risultante viene tradotto in
istruzioni testuali (`src/ai/toneEngine.js`) incluse nel system prompt inviato a Grok.

Puoi azzerare in ogni momento il tono che Ranji ha imparato su di te con `/resettone`.

## 🔧 Personalizzazione

- **Cambiare provider AI**: sostituisci `src/ai/grokClient.js`. L'SDK usato (`openai`)
  è compatibile con qualsiasi endpoint che rispetti lo schema OpenAI Chat Completions.
- **Aggiungere un comando**: crea un file in `src/commands/moderation/` o `utility/`
  che esporti `{ data, execute }`, poi rilancia `npm run deploy`.
- **Aggiungere un evento**: crea un file in `src/events/` che esporti `{ name, once?, execute }`.
- **Filtro parole/automod**: punto di partenza consigliato è estendere
  `src/events/messageCreate.js` con un controllo su una lista di parole vietate per server.

## 📄 Licenza

MIT — vedi `LICENSE`. Usalo, modificalo, ospitalo dove vuoi.
