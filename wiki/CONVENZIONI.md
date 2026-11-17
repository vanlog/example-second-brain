# Convenzioni del wiki

Regole di struttura e formato per tutte le note di `wiki/`. Le regole generali (sola lettura di `raw/`, niente duplicati, fonti obbligatorie) sono in `CLAUDE.md`.

## Struttura delle cartelle

```
wiki/
├── 00-indice.md              punto di ingresso: link agli indici di cartella + timeline + log
├── CONVENZIONI.md            questo file
├── log.md                    registro append-only degli aggiornamenti
├── clienti/                  cliente-<slug>.md
├── contratti/                contratto-<cliente>-<tipo>.md
├── offerte/                  offerta-<anno>-<n>.md
├── progetti/                 progetto-<slug>.md
├── fatture/                  fattura-<numero>.md
├── dlv/                      dlv-<nnn>-<slug>.md
├── call/                     call-<aaaa-mm-gg>-<slug>.md
├── persone/                  persona-<nome-cognome>.md
├── timeline/                 timeline-<anno>.md
└── incongruenze/             incongruenza-<slug>.md
```

`_staging/` contiene le schede di lavoro dell'import (`_staging/<cliente>/<offerta-AAAA-N o _generale>/<fonte>--<file>.md`, più `_interno/` e `_azienda/`): **non sono note**, non vanno indicizzate né sottoposte a lint, e hanno un solo registro (`_staging/_registro-staging.md`, sha256 dei file raw). Le cartelle di entità non hanno più un registro proprio.

Ogni cartella ha un indice `_indice-<cartella>.md` (es. `_indice-clienti.md`): elenco delle note con link e una riga di sintesi. Il nome include la cartella perché in Obsidian `[[_indice]]` sarebbe ambiguo. `00-indice.md` linka gli indici di cartella e non ne ripete il contenuto.

## Slug e nomi dei file

- Minuscolo, trattini, niente accenti né `&` (`Ferro & Forma` → `ferro-e-forma`).
- Prefisso del tipo sempre presente: lo slug è anche il nome del file e il target del wikilink.
- Il numero della fattura va in minuscolo: `2026-1C` → `fattura-2026-1c`. Nel campo `numero` resta come in origine.
- Le persone usano nome e cognome: `persona-laura-conti`.

## Frontmatter comune

Tutte le note di entità iniziano con:

```yaml
---
tipo: cliente | contratto | offerta | progetto | fattura | dlv | call | persona | incongruenza
stato: <valori ammessi per tipo, vedi sotto>
fonti:
  - raw/connettori/...   # percorsi completi, almeno uno
ultimo_aggiornamento: AAAA-MM-GG
---
```

- `ultimo_aggiornamento` e le date di `log.md` usano la **data di riferimento** di `CLAUDE.md` (oggi 2026-07-03), non la data di sistema.
- I campi **derivati** (calcolati, non letti da una fonte) sono elencati sotto con `# derivato` e si ricalcolano a ogni aggiornamento. Non si scrivono in nessun'altra nota.

## Tipi di nota

### cliente — `clienti/cliente-<slug>.md`
```yaml
stato: attivo | prospect | ex-cliente
ragione_sociale:
partita_iva:
settore:
referente: "[[persona-...]]"
account: "[[persona-...]]"          # account manager Musa AI
ultimo_contatto:                    # derivato: data più recente tra email e call
```
Corpo: profilo (2-3 righe), link a contratti, offerte, progetti e call. Niente importi: si trovano nelle offerte e nelle fatture.

### contratto — `contratti/contratto-<cliente>-<tipo>.md`
`tipo`: `quadro`, `progetto`, `retainer`, `sow`.
```yaml
stato: attivo | scaduto | disdetto
cliente: "[[cliente-...]]"
data_firma:
durata:
rinnovo: automatico | nessuno
preavviso_disdetta:                 # es. 60 giorni
termine_disdetta:                   # data ultima utile per la disdetta (fonte: calendario)
esclusiva: si | no
```

### offerta — `offerte/offerta-<anno>-<n>.md`
```yaml
stato: inviata | accettata | rifiutata | scaduta | assorbita
cliente: "[[cliente-...]]"
data:
valore:                             # € IVA esclusa
condizioni_pagamento:               # tranche e scadenze
avvio_previsto:
fine_prevista:
scadenza_offerta:                   # se presente (fonte: calendario)
progetto: "[[progetto-...]]"        # solo se accettata
```
- Il contenuto della proposta viene dal documento Drive, lo **stato** dalle email di risposta (`Re: Offerta ...`): Drive non traccia lo stato commerciale.
- `assorbita` vale quando l'offerta non viene firmata ma l'attività confluisce in un accordo esistente (es. 2026-6 inclusa nel retainer); va spiegato nel corpo.

### progetto — `progetti/progetto-<slug>.md`
Esiste solo per un'offerta accettata (o un contratto retainer).
```yaml
stato: in-corso | completato | sospeso
cliente: "[[cliente-...]]"
offerta: "[[offerta-...]]"
contratto: "[[contratto-...]]"
avvio:
fine_prevista:                      # fonte: calendario "fine prevista progetto"
team: ["[[persona-...]]"]
fatturato_emesso:                   # derivato: somma delle fatture collegate
residuo_da_fatturare:               # derivato: valore offerta − fatturato_emesso
stato_calcolato_oggi:               # derivato: es. "in ritardo su DLV-019"
```
Corpo: obiettivo in una riga, tabella dei DLV (solo link e stato), tabella delle fatture (solo link). Il valore e le condizioni restano nell'offerta.

### fattura — `fatture/fattura-<numero>.md`
```yaml
stato: in-attesa | pagata | scaduta  # come da export Fatture in Cloud
numero: 2026-1C
cliente: "[[cliente-...]]"
offerta: "[[offerta-...]]"           # dal numero: <anno>-<offerta><lettera>
data_emissione:
importo:
scadenza_pagamento:
stato_calcolato_oggi:                # derivato: es. "scaduta da 12 giorni"
```

### dlv — `dlv/dlv-<nnn>-<slug>.md`
```yaml
stato: pianificato | consegnato | approvato | in-ritardo
progetto: "[[progetto-...]]"
scadenza:                           # fonte: evento calendario [DLV-nnn]
owner: "[[persona-...]]"             # organizzatore dell'evento
data_consegna:                      # fonte: email di consegna ("consegna", "attivo", "versioni definitive")
```
`in-ritardo` è derivato: scadenza passata rispetto a oggi e nessuna email di consegna.

### call — `call/call-<aaaa-mm-gg>-<slug>.md`
Una nota per riunione, anche se compare sia nel calendario sia in Granola: entrambi i file vanno in `fonti`.
```yaml
stato: svolta | pianificata | provvisoria
data:
partecipanti: ["[[persona-...]]"]
cliente: "[[cliente-...]]"           # assente per riunioni interne
progetto: "[[progetto-...]]"
```
Corpo: 3-6 punti su **decisioni** e **azioni** (chi, cosa, entro quando). Niente trascrizione.

### persona — `persone/persona-<nome-cognome>.md`
```yaml
stato: attiva | uscita
organizzazione: Musa AI | "[[cliente-...]]"
ruolo:
email:
dal:                                # data di ingresso, se nota
```
Corpo: responsabilità e link a clienti e progetti seguiti.

### incongruenza — `incongruenze/incongruenza-<slug>.md`
```yaml
stato: aperta | chiusa
note_coinvolte: ["[[...]]"]
fonti: [...]                        # tutte le fonti in conflitto
```
Corpo: il fatto, i valori in conflitto con la fonte di ciascuno, e la risoluzione (solo dopo la conferma dell'utente).

Esempio: l'email `2026-06-25_0940_sollecito-pagamento-fattura-ft-2026-028.md` cita la fattura **FT-2026-028**. Quel numero non rispetta la numerazione di Fatture in Cloud (`<anno>-<offerta><lettera>`) e non c'è nell'export → `incongruenza-fattura-ft-2026-028`.

## Mappa: fatto → nota proprietaria

Ogni fatto si scrive **solo** qui; le altre note linkano.

| Fatto | Nota proprietaria | Fonte principale |
|---|---|---|
| Ragione sociale, P.IVA, settore, referente | cliente | fatture-in-cloud/clienti, Drive |
| Nome, ruolo, email di una persona | persona | email (firme), calendario |
| Durata, rinnovo, disdetta, esclusiva | contratto | Drive/Contratti, calendario |
| Contenuto, valore, condizioni, tempistiche proposte | offerta | Drive/Offerte |
| Stato commerciale dell'offerta | offerta | email `Re: Offerta ...` |
| Avvio, fine prevista, team, stato del progetto | progetto | email di kickoff, calendario |
| Importo, date e stato di una fattura | fattura | fatture-in-cloud/fatture |
| Scadenza e owner di un deliverable | dlv | calendario `[DLV-nnn]` |
| Avvenuta consegna di un deliverable | dlv | email di consegna |
| Decisioni e azioni di una riunione | call | Granola, calendario |
| Eventi aziendali (assunzioni, traguardi) | timeline | email, Drive/Azienda |
| Conflitti tra fonti | incongruenza | — |

Le email **non** hanno note proprie: sono fonti dei fatti sopra.

## Dati sensibili

Nelle note **non** si riportano: numeri di telefono, indirizzi, link di videoconferenza, contenuto integrale di email o trascrizioni. Restano ammessi la P.IVA (nella nota cliente) e l'email professionale (nella nota persona).

## Timeline

`timeline/timeline-<anno>.md`: un elenco cronologico di righe `- AAAA-MM-GG — evento — [[nota]]`. Contiene solo link a note esistenti e fatti già presenti lì, mai fatti nuovi.

## log.md

Append-only, una riga per aggiornamento, nel formato:

```
- AAAA-MM-GG | ingest|query|lint|correzione | cosa è cambiato | note: [[...]], [[...]] | fonti: raw/connettori/...
```

## Corpo delle note

Il corpo deve essere **completo e autosufficiente**: chi legge la nota non deve aprire `raw/` per capire la situazione. Non c'è un limite fisso di righe; il limite è la ridondanza (un fatto vive in una sola nota, le altre linkano) e i dati sensibili. Le indicazioni "una riga" o "3-6 punti" nelle sezioni dei tipi sopra sono un minimo, non un massimo.

Ogni nota ha, dopo il frontmatter, queste sezioni (omettere solo quelle senza contenuto):

| Tipo | Sezioni del corpo |
|---|---|
| cliente | Profilo · Storia della relazione (date, offerte, progetti, momenti chiave) · Referenti e dinamiche · Stato commerciale e contrattuale (solo link) · Rischi e punti aperti · Decisioni e fatti rilevanti |
| contratto | Oggetto e perimetro · Condizioni (durata, rinnovo, preavviso, esclusiva, penali) · Storia (firme, rinnovi, modifiche) · Scadenze e azioni · Decisioni e fatti rilevanti |
| offerta | Contenuto della proposta (attività, deliverable) · Condizioni e tempistiche · Iter commerciale (invio, risposte, trattative, esito) · Collegamenti (progetto, fatture) · Decisioni e fatti rilevanti |
| progetto | Obiettivo e perimetro · Team e ruoli · Avanzamento (cosa è stato fatto, cosa manca) · Tabella DLV (link e stato) · Tabella fatture (link) · Rischi e ritardi · Decisioni e fatti rilevanti |
| fattura | Cosa fattura (tranche, riferimento a offerta/progetto) · Stato e scadenza · Solleciti e comunicazioni · Decisioni e fatti rilevanti |
| dlv | Cosa è il deliverable · Scadenza, owner, consegna · Revisioni e feedback del cliente · Dipendenze e rischi · Decisioni e fatti rilevanti |
| call | Contesto e partecipanti · Argomenti discussi · Decisioni · Azioni (chi, cosa, entro quando) · Punti aperti |
| persona | Ruolo e responsabilità · Clienti e progetti seguiti · Storia (ingresso, cambi di ruolo, uscita) · Decisioni e fatti rilevanti |
| timeline | Righe cronologiche come da sezione Timeline |

**Decisioni e fatti rilevanti**: elenco `- AAAA-MM-GG — fatto o decisione — fonte` con le date, le decisioni, le richieste del cliente, le obiezioni, i ritardi e le azioni aperte che emergono dalle fonti. Ogni riga ha la fonte; ciò che è dedotto è marcato "dedotto". Il fatto proprietario di un'altra nota (importi, scadenze, valori) non si ripete: si linka.
