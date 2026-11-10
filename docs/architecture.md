# Marea Lead Finder: specifica v1

Stato: approvata. Sostituisce la specifica "Marea OS fase 0.1" (troppo ampia per l'obiettivo).

## Obiettivo

Ogni giorno, una lista di attività della provincia di Cuneo da contattare, ordinate per probabilità di comprare un sito, senza mai riproporre chi è già stato contattato.

## Cosa fa

1. **Ricerca:** per ogni coppia nicchia × comune (es. "parrucchiere Savigliano") interroga Google Places API (New), Text Search, con paginazione.
2. **Filtri** (tutti in `config.yaml`, nessuno fisso nel codice):
   - CAP che inizia con `12` (provincia di Cuneo);
   - almeno `min_reviews` recensioni (iniziale: 15);
   - attività operativa;
   - sito: nessun sito o solo social (Instagram, Facebook, Linktree, TikTok, wa.me, Google Sites) → tenuta; sito vero → esclusa o messa in fondo (configurabile).
3. **Priorità:**
   1. nessun sito > solo social > sito vero;
   2. cellulare (`+39 3…`) prima del fisso;
   3. a parità, più recensioni prima.
4. **Deduplica:** per `place_id` e per telefono normalizzato (E.164), sia dentro i risultati sia contro lo storico. Un numero condiviso da due attività le fa comparire una volta sola.
5. **Output:** `output/leads_AAAA-MM-GG.csv` con: priorità, nome, categoria, comune, telefono, cellulare sì/no, sito, recensioni, rating, link Google Maps, link `wa.me`.
6. **Storico:** `contattati.csv` (data, place_id, telefono, canale, esito). Lo aggiorni tu. Chi è nello storico non ricompare mai; chi ha esito `no` / `non_ricontattare` è escluso per sempre.

## Stack

- Python 3, un solo script (`lead_finder.py`) + `config.yaml`.
- Chiave Google in variabile d'ambiente `GOOGLE_PLACES_API_KEY`, mai nel repo.
- Nessun database, nessun server, nessuna app web.

## Costi

- Lo script conta le richieste a Google e le stampa a fine esecuzione.
- Su Google Cloud: chiave limitata a Places API (New) e avviso di budget.

## Fuori dalla v1

CRM, dashboard, follow-up automatici, app web, invii automatici, AI.
Si aggiungono solo se il lead finder viene usato ogni giorno per almeno 2 settimane e il collo di bottiglia diventa la gestione, non la ricerca.

## Rischi accettati

- I termini di Google limitano la conservazione dei dati Places (nel SEE si può tenere a tempo indeterminato solo il `place_id`). Salvare i lead in CSV è una scelta del titolare.
- Il primo contatto a freddo via WhatsApp/email verso attività è regolato dall'art. 130 del Codice Privacy. Scelta del canale a carico del titolare.

## Test di accettazione

- **AT-1 Telefono:** `+39 333 123 4567`, `0039 3331234567`, `333-123-4567` → `+393331234567` (cellulare); `0172 123456` → `+390172123456` (fisso); `12345` → scartato.
- **AT-2 Sito:** `instagram.com/x`, `m.facebook.com/x`, `linktr.ee/x`, `sites.google.com/view/x` → solo social; `centro-bellezza.it` → sito vero; assente → nessun sito.
- **AT-3 Area:** `12038 Savigliano CN` → dentro; `10100 Torino TO` → fuori.
- **AT-4 Duplicati:** stessa attività trovata da due ricerche → una riga. Due attività con lo stesso numero → una riga.
- **AT-5 Storico:** un `place_id` o un telefono presente in `contattati.csv` non compare nell'output.
- **AT-6 Ordine:** nessun sito + cellulare > solo social + cellulare > nessun sito + fisso > sito vero.
- **AT-7 Config:** cambiando `min_reviews` da 15 a 30 nel config, l'output cambia senza toccare il codice.
- **AT-8 Prova reale:** una ricerca vera in provincia di Cuneo; controllo a mano di 10 righe.
