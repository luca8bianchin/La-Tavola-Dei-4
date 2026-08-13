# La Tavola dei 4 — Guida al setup

L'app è **già funzionante**: aprendo `index.html` puoi votare subito. In modalità
"solo locale" i dati restano salvati **su questo telefono** (non si perdono quando
chiudi l'app) ma non sono condivisi tra amici.

Per avere i **voti condivisi + live** tra Luca, Arianna, Asia ed Elia servono 2 cose:
1. Un database **Supabase** (gratis)
2. Mettere l'app **online** con un link (per condividerla su WhatsApp)

Tempo totale: ~10 minuti.

---

## PARTE 1 — Database Supabase (voti condivisi)

### 1. Crea il progetto
1. Vai su **https://supabase.com** → *Start your project* → accedi con Google/GitHub.
2. *New project* → dai un nome (es. `tavola-dei-4`), scegli una password qualsiasi
   e la region più vicina (es. *West EU / Frankfurt*). Attendi ~1 minuto.

### 2. Crea le tabelle
Nel menu a sinistra apri **SQL Editor** → *New query*, incolla **tutto** questo e premi **Run**:

```sql
-- Ristoranti
create table restaurants (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  country text,
  created_at timestamptz default now()
);

-- Serate (una visita = una sessione di voti)
create table sessions (
  id uuid primary key default gen_random_uuid(),
  restaurant_id uuid references restaurants(id) on delete cascade,
  date date default current_date,
  created_by text,
  created_at timestamptz default now()
);

-- Voti (uno per amico per serata)
create table votes (
  id uuid primary key default gen_random_uuid(),
  session_id uuid references sessions(id) on delete cascade,
  restaurant_id uuid references restaurants(id) on delete cascade,
  user_name text not null,
  location numeric,
  menu numeric,
  prezzo numeric,
  servizio numeric,
  special numeric,
  special_note text,
  created_at timestamptz default now(),
  unique (session_id, user_name)
);

-- Permessi: attiviamo RLS e diamo accesso pubblico (siete un gruppo privato di amici)
alter table restaurants enable row level security;
alter table sessions   enable row level security;
alter table votes      enable row level security;

create policy "public read"  on restaurants for select using (true);
create policy "public write" on restaurants for insert with check (true);
create policy "public read"  on sessions   for select using (true);
create policy "public write" on sessions   for insert with check (true);
create policy "public read"  on votes       for select using (true);
create policy "public write" on votes       for insert with check (true);
create policy "public update" on votes      for update using (true) with check (true);
```

> Nota: le policy sono "aperte" perché siete un piccolo gruppo privato e usate la
> chiave *anon* (non quella segreta). Chi non ha il link non trova nulla. Se in
> futuro vuoi più sicurezza, si aggiunge un login — dimmelo e te lo preparo.

### 3. Copia le chiavi
Menu a sinistra → **Project Settings** (ingranaggio) → **API**. Copia:
- **Project URL** → es. `https://abcdxyz.supabase.co`
- **anon public** key → una stringa lunga che inizia con `eyJ...`

### 4. Incolla le chiavi nell'app
Apri `index.html` con un editor di testo (Note, VSCode, ecc.), cerca in alto
queste due righe e incolla i tuoi valori tra le virgolette:

```js
const SUPABASE_URL  = "https://abcdxyz.supabase.co";
const SUPABASE_ANON = "eyJhbGciOi....la-tua-chiave....";
```

Salva. Fatto: ora l'app usa il database condiviso. In alto vedrai il pallino
verde **"Database condiviso live"**.

---

## PARTE 2 — Mettere l'app online (link WhatsApp)

Ti serve un URL da mandare agli amici. Il modo più semplice e gratuito:

### Opzione A — Netlify Drop (più veloce, 30 secondi)
1. Vai su **https://app.netlify.com/drop**
2. Trascina il file `index.html` (o l'intera cartella) nella pagina.
3. Netlify ti dà un link tipo `https://tavola-dei-4.netlify.app`. Quello è il link
   da condividere. Puoi rinominarlo dalle impostazioni del sito.

### Opzione B — GitHub Pages
1. Crea un repository su GitHub, carica `index.html`.
2. *Settings → Pages → Deploy from branch → main → /root*.
3. Dopo ~1 minuto avrai `https://tuonome.github.io/nome-repo/`.

Quando aggiorni le chiavi Supabase, ricarica il file aggiornato sullo stesso servizio.

---

## Come si usa

- **Chi apre il link** sceglie chi è (Luca / Arianna / Asia / Elia): da quel momento
  l'app registra i voti a nome suo su quel telefono.
- **Nuova serata** → scegli un ristorante *già provato* dalla lista scrollabile,
  oppure **Nuovo ristorante** (nome + paese). Uno stesso ristorante può avere più serate.
- Ognuno mette i suoi voti (1–10, step 0,5) sulle 5 categorie + la nota **Special**
  scritta a mano.
- **Tasto Aggiorna** (icona ⟳ in home): mentre votate insieme, premilo per vedere
  i voti degli altri in tempo reale.
- **Classifica**: ristoranti ordinati per media totale (oro = migliore).
  **Ultime visite**: ordinati per data.
- **Tocca una card** → resoconto completo: media totale, medie per categoria,
  e ogni **serata** con la sua media e tutti i voti dei singoli amici + le note special.

### Come si calcola il totale
Media di **tutti** i voti di **tutte** le serate (ogni voto = media delle sue 5 categorie).
Nel resoconto vedi anche la **media di ogni singola serata**, come richiesto.

---

Se preferisci, posso automatizzarti il deploy o aggiungere un login vero.
Dimmi solo come vuoi procedere.
