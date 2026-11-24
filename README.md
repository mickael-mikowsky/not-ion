# Not(ion)

Un clone **local et privé** de l'interface Notion, dans un seul fichier `index.html`.
Pas de compte, pas de serveur, pas d'IA, pas de collaboration : tes pages restent dans ton navigateur.

## Mise en ligne sur GitHub Pages

1. Crée un dépôt (par ex. `notion-perso`) et mets-y **uniquement** `index.html`, `README.md` et `.gitignore`.
2. Dans le dépôt : Settings → Pages → Source : `Deploy from a branch` → branche `main`, dossier `/ (root)`.
3. Ouvre `https://<ton-pseudo>.github.io/notion-perso/`.

> ⚠️ Ne mets jamais `notion-import.json` ni tes sauvegardes dans le dépôt : un dépôt GitHub Pages est public. Le `.gitignore` les exclut.
> L'app hébergée ne contient que le code. Tes données restent dans le navigateur qui l'ouvre.

Tu peux aussi ouvrir `index.html` directement depuis ton disque, sans hébergement.

## Importer tes données (une seule fois)

Importer (en bas de la barre latérale) → `notion-import.json`. Tu peux aussi glisser le fichier sur la fenêtre.
Ça crée la base **To do Boulot IA** avec ses 267 entrées et 3 vues : Kanban par étiquette, Kanban par état, Tableau.

L'app sait aussi importer un export Notion `.zip` (Markdown & CSV), ou des fichiers `.md` / `.csv`.

## Stockage en ligne (Supabase)

Tes pages sont **chiffrées dans le navigateur** avant d'être envoyées : Supabase ne stocke qu'un bloc illisible.

1. Sur supabase.com, crée un projet (le plan gratuit suffit).
2. **SQL Editor** : colle le script ci-dessous et clique sur **Run**.
3. **Authentication → Users → Add user** : crée ton compte (email et mot de passe).
4. **Authentication → Sign In / Providers → Email** : désactive « Allow new users to sign up », pour que personne d'autre ne puisse créer de compte.
5. **Project Settings → API** : copie la « Project URL » et la clé « anon public ».
6. Sur GitHub, ouvre `index.html` (icône crayon) et cherche `CLOUD_URL`. Colle l'URL et la clé entre les apostrophes, puis enregistre (Commit) :
   ```js
   const CLOUD_URL='https://xxxx.supabase.co';
   const CLOUD_ANON='eyJhbGciOi…';
   ```
   Tu peux aussi les saisir dans Paramètres → Stockage en ligne, mais il faudra les ressaisir après chaque vidage du cache.
7. Ouvre l'app et connecte-toi. Si tes données sont encore dans le navigateur, elles partent en ligne dès que tu définis un **mot de passe de chiffrement** (Paramètres → Sécurité).

À chaque ouverture : connexion Supabase (seulement si le cache a été vidé), puis mot de passe de chiffrement.
Indicateur en haut à droite : « ☁ En ligne », « ☁ Synchro… » ou « ☁ Hors ligne » (l'app réessaie automatiquement).

> La clé `anon` peut être publique, c'est son rôle. Ce sont les règles RLS du script qui empêchent quiconque d'autre de lire ta ligne.
> Mot de passe de chiffrement perdu = données irrécupérables. Exporte une sauvegarde de temps en temps.

```sql
create table if not exists public.notion_state (
  user_id uuid primary key default auth.uid() references auth.users(id) on delete cascade,
  payload jsonb not null,
  updated_at timestamptz not null default now()
);
alter table public.notion_state enable row level security;
create policy "lire ses donnees" on public.notion_state for select using (auth.uid() = user_id);
create policy "creer ses donnees" on public.notion_state for insert with check (auth.uid() = user_id);
create policy "modifier ses donnees" on public.notion_state for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

## Sécurité

- La seule connexion réseau autorisée est vers `*.supabase.co`, pour le stockage en ligne. Tout le reste est bloqué par la politique de sécurité de la page.
- Stockage dans IndexedDB, propre à ce navigateur et à cette adresse.
- Paramètres → Sécurité : mot de passe optionnel, qui chiffre toutes les données (PBKDF2 + AES-256-GCM). Si tu perds le mot de passe, les données sont perdues.
- Fais une sauvegarde de temps en temps (Paramètres → Exporter une sauvegarde) : vider les données du navigateur efface tout.

## Raccourcis principaux

| Action | Raccourci |
| --- | --- |
| Menu des blocs | `/` |
| Rechercher | `Ctrl+P` ou `Ctrl+K` |
| Barre latérale | `Ctrl+\` |
| Mode sombre | `Ctrl+Maj+L` |
| Nouvelle page | `Ctrl+Alt+N` |
| Lier une page | `[[` ou `@` |
| Transformer un bloc | `Ctrl+Maj+0…9` (`⌘+⌥+0…9` sur Mac) |
| Dupliquer / déplacer | `Ctrl+D` / `Ctrl+Maj+↑↓` |
| Sélectionner le bloc | `Échap` |
| Annuler | `Ctrl+Z` |

Markdown en début de ligne : `#`, `##`, `###`, `-`, `1.`, `[]`, `>` (toggle), `"` (citation), ` ``` `, `---`.
Liste complète : bouton **?** en bas à droite.
