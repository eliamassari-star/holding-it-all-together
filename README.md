# Holding It All Together

Sito editoriale responsive e redazione AI per **Holding It All Together — Work. Money. Family. Life.** Comprende homepage, articoli e categorie, newsletter con archivio, podcast con episodi e feed RSS, ricerca, iscrizione, pagine legali e uno Studio privato che coordina otto agenti specializzati.

Il modello operativo della redazione è descritto in [`docs/redazione-ai.md`](docs/redazione-ai.md).

## Avvio locale

Estrai lo ZIP e apri un terminale nella cartella `holding-it-all-together`.
Servono Node.js 22.13 o successivo, pnpm e una connessione al registro npm per
installare le dipendenze. La configurazione locale usa Vinext e il simulatore
Cloudflare D1. Puoi avviare il sito senza un account Sites o Cloudflare.

```bash
pnpm install --frozen-lockfile
cp .env.example .dev.vars
```

Su Windows puoi copiare il file con `Copy-Item .env.example .dev.vars` in PowerShell.
In `.dev.vars` imposta `NEXT_PUBLIC_SITE_URL=http://localhost:3000` e lascia
`NEWSLETTER_PROVIDER=mock`. Poi avvia il server locale.

```bash
pnpm dev --host 127.0.0.1 --port 3000
```

Apri [la homepage](http://localhost:3000), [lo Studio](http://localhost:3000/studio)
o [il feed podcast](http://localhost:3000/podcast/rss.xml).
Il database locale viene inizializzato dall'app. L'iscrizione di prova registra
il consenso in locale senza inviare email. Puoi controllare il progetto con
`pnpm typecheck` e generare la build Worker con `pnpm build`.

Lo ZIP è basato sulla versione 12 del sorgente Sites, commit
`765c4ca011738e5306126e4e98507dd69f9d2291`. Include gli asset del repository,
il lockfile e lo schema D1. Il database locale parte vuoto e usa il catalogo
incluso in `app/data/content.ts`. I dati D1 di produzione e le configurazioni
riservate dei servizi esterni sono fuori dal perimetro dell'esportazione.

Senza `OPENAI_API_KEY`, lo Studio usa una modalità dimostrativa completa e deterministica. Con la chiave configurata utilizza la Responses API e la ricerca web per gli agenti Scout e Ricercatore.

## Studio e agenti

Il flusso mantiene sempre l'approvazione umana prima della pubblicazione:

1. **Scout argomenti** propone cinque idee attuali.
2. **Caporedattore** prepara il brief.
3. **Ricercatore** crea il dossier con fonti.
4. **Autore** scrive articolo e metadata.
5. **Produttore podcast** prepara episodio, clip e pacchetto YouTube/Spotify.
6. **Revisore** controlla accuratezza, tono e coerenza.
7. **Site Manager e Analista** completa il controllo operativo.
8. Il proprietario approva i canali; il **Publisher** pubblica sul sito e prepara i pacchetti esterni.

Ogni esecuzione, artefatto, fonte, versione, approvazione ed evento di pubblicazione viene registrato nel database D1. Le route `/api/studio/*` sono private in produzione: configura `EDITOR_USER_IDS` o `EDITOR_EMAILS` con gli account autorizzati.

## Pubblicazione

Il progetto usa Vinext e OpenAI Sites su un runtime Cloudflare Worker con binding D1 `DB`.

Prima di rendere pubblico il sito:

- sostituisci `NEXT_PUBLIC_SITE_URL` con il dominio definitivo;
- configura `OPENAI_API_KEY` e, se necessario, `OPENAI_EDITORIAL_MODEL`;
- limita lo Studio con `EDITOR_USER_IDS` o `EDITOR_EMAILS`;
- imposta gli URL social e podcast reali;
- completa titolare e contatti nelle pagine legali;
- collega il provider newsletter scelto;
- collega `EDITORIAL_PUBLISH_WEBHOOK_URL` per consegnare automaticamente i pacchetti a hosting podcast e strumenti di distribuzione.

## Contenuti

I nuovi contenuti pubblicati risiedono in D1. `app/data/content.ts` contiene il catalogo dimostrativo e funge da fallback finché non viene completato il primo ciclo reale.

- Per il flusso ordinario usa `/studio`: i contenuti vengono validati, versionati e pubblicati senza modificare il codice.
- Per modificare i contenuti dimostrativi, aggiorna gli array in `app/data/content.ts`.

Le pagine dinamiche, i metadati SEO, la sitemap e il feed vengono generati automaticamente dai dati.

## Immagini

Gli asset principali sono in `public/images`:

- `home-hero.png`: hero homepage e immagini famiglia;
- `newsletter-hero.png`: still life della newsletter;
- `newsletter-cta.png`: banner panoramico;
- `podcast-hero.png`: hero verticale podcast.

Per sostituirli, mantieni gli stessi nomi oppure aggiorna i percorsi in `app/data/content.ts`. Usa immagini ad alta risoluzione e conserva le proporzioni indicate dal layout; i crop sono gestiti con `object-fit: cover`.

## Newsletter

Il form include validazione email, consenso privacy, honeypot anti-spam, limite temporaneo dei tentativi, stato di caricamento, successo ed errore. Il provider si sceglie con `NEWSLETTER_PROVIDER`:

- `mock`: flusso locale completo senza invio esterno;
- `beehiiv`: richiede `BEEHIIV_API_KEY` e `BEEHIIV_PUBLICATION_ID`;
- `brevo`: richiede `BREVO_API_KEY` e `BREVO_LIST_ID`; per il double opt-in configura anche `BREVO_DOI_TEMPLATE_ID` e `BREVO_DOI_REDIRECT_URL`;
- `mailchimp`: richiede `MAILCHIMP_API_KEY`, `MAILCHIMP_SERVER_PREFIX` e `MAILCHIMP_LIST_ID`; i nuovi contatti vengono creati con stato `pending`.

Le pagine preferenze e disiscrizione funzionano localmente; in produzione possono inoltrare le richieste a un endpoint proprietario tramite `NEWSLETTER_ACCOUNT_WEBHOOK_URL` e `NEWSLETTER_ACCOUNT_WEBHOOK_TOKEN`. Il consenso, compreso quello separato per il tracciamento individuale, è registrato in D1 insieme alla versione dell'informativa. `NEWSLETTER_SIGNING_SECRET` abilita l'endpoint RFC 8058 `/api/newsletter/unsubscribe-one-click` da usare negli header `List-Unsubscribe` e `List-Unsubscribe-Post` del provider.

Le credenziali non devono mai essere inserite nel codice o in variabili con prefisso `NEXT_PUBLIC_`.

## Podcast e social

I link sono centralizzati in `app/config.ts` e configurabili con:

- `NEXT_PUBLIC_PODCAST_RSS_URL`
- `NEXT_PUBLIC_SPOTIFY_URL`
- `NEXT_PUBLIC_APPLE_PODCASTS_URL`
- `NEXT_PUBLIC_YOUTUBE_URL`
- `NEXT_PUBLIC_INSTAGRAM_URL`
- `NEXT_PUBLIC_LINKEDIN_URL`

Il feed RSS locale è disponibile su `/podcast/rss.xml` e include metadata iTunes, trascrizioni VTT e enclosure quando l'episodio possiede un `audioUrl`. Per usare un feed esterno basta impostare `NEXT_PUBLIC_PODCAST_RSS_URL`.

Il pacchetto editoriale segue una strategia video-first: episodio completo 16:9 nativo per YouTube e Spotify, versione fruibile anche solo in audio, capitoli e trascrizione manuali, una clip verticale autonoma per episodio e un'unica CTA. Lo Studio mostra il playbook aggiornato e i collegamenti alle fonti ufficiali.

## Variabili principali

- `OPENAI_API_KEY`, `OPENAI_EDITORIAL_MODEL`: generazione editoriale live;
- `EDITOR_USER_IDS`, `EDITOR_EMAILS`: accesso allo Studio;
- `EDITORIAL_PUBLISH_WEBHOOK_URL`, `EDITORIAL_PUBLISH_WEBHOOK_TOKEN`: consegna del pacchetto multicanale;
- `NEWSLETTER_PROVIDER` e credenziali del provider: iscrizioni double opt-in;
- `NEWSLETTER_PRIVACY_VERSION`, `NEWSLETTER_SIGNING_SECRET`: consenso e disiscrizione one-click;
- `PODCAST_OWNER_EMAIL`: proprietario del feed RSS;
- `NEXT_PUBLIC_*`: dominio e collegamenti pubblici.

## Struttura principale

- `app/components`: componenti riutilizzabili per header, footer, card, newsletter, piattaforme e form;
- `app/data`: contenuti editoriali strutturati;
- `app/studio` e `app/api/studio`: interfaccia privata e workflow degli agenti;
- `lib/editorial`: agenti, Responses API, playbook, repository e contenuti pubblici;
- `db` e `drizzle`: schema D1 e migrazione;
- `lib/newsletter.ts`: adapter dei provider newsletter;
- `app/api`: endpoint per iscrizione, preferenze e disiscrizione;
- `app/globals.css`: design system derivato dal mockup, layout e breakpoint;
- `app/sitemap.ts` e `app/robots.ts`: SEO tecnico.

## Modifiche editoriali della versione GitHub (novembre 2026)

- Homepage ripensata per raccontare la vita reale, anziché promettere una vita perfetta.
- Manifesto e percorso narrativo con accesso a storie ed episodi.
- Testo della pagina Chi siamo coerente con la visione del progetto.
- Miglioramenti all'accessibilità (salta al contenuto, movimento ridotto) e al layout mobile.
- URL pubblico predefinito aggiornato al dominio condiviso dal proprietario.

**Nota importante:** i contenuti del catalogo iniziale sono dimostrativi. Non presentarli come testimonianze autobiografiche finché non vengono sostituiti con storie realmente pubblicate. Il codice contiene servizi di newsletter, Studio AI e database D1: per distribuirli fuori da Sites occorre configurare hosting, migrazioni e segreti secondo `.env.example`.
