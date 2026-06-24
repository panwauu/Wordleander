# Wordleander — Requirements

## Overview
A single-page web app where users collaboratively collect and vote on words using a star rating and a three-dimensional barycentric score.

Deployed via GitHub Pages. Backend is Supabase (anonymous REST API with a shared secret token guard).

## Access Control
- The page is protected by a `?token=<SECRET>` URL parameter.
- Without the token, the page renders blank.
- There is no real authentication — the username is self-reported and stored as plain text.

## User Flow
1. User arrives at `index.html?token=<SECRET>`.
2. User enters a display name (free text, no validation against existing names).
3. Name is stored in JS state only (no cookie/localStorage persistence — logout resets everything).
4. Main app is revealed with four tabs.

## Tabs

### + Hinzufügen (Add)
- Enter a word (text input).
- Select a general star rating (1–5).
- Click inside the triangle to set dimensional scores.
- Submit → creates one row in `words` and one row in `ratings` (the creator's own vote).

### Bewerten (Rate)
- Lists all words the current user did **not** add and has **not yet rated**.
- Clicking "Bewerten" opens an inline modal with the same star + triangle UI.
- Submitting saves a row to `ratings`.

### Alle Wörter (List)
- Lists every word with: added-by, vote count, average stars, average dimension breakdown.
- Rows expand to show per-user rating details.
- Badges indicate if the current user added or already rated the word.

### Dreieck (Triangle Viz)
- Draws all words as dots on the barycentric triangle using their average dimension scores.
- Hover over a dot shows the word name, average stars, and dimension percentages.

## Voting Model

### Stars
- Integer 1–5, required for both adding and rating.

### Triangle (Dimensions)
- Three dimensions: **atzig**, **fotzig**, **mausig**.
- Scores are barycentric coordinates — they always sum to 1.
- User clicks inside the equilateral triangle; the click position is converted to (atzig, fotzig, mausig) coordinates.
- Stored as three separate numeric columns in Supabase.

## Database Schema (Supabase / PostgreSQL)

```sql
create table words (
  id text primary key,        -- 'w' + timestamp + random suffix
  word text not null,
  added_by text not null,
  created_at timestamptz default now()
);

create table ratings (
  id uuid primary key default gen_random_uuid(),
  word_id text references words(id),
  username text not null,
  stars integer not null,      -- 1–5
  atzig numeric not null,      -- barycentric, sum with fotzig+mausig = 1
  fotzig numeric not null,
  mausig numeric not null,
  created_at timestamptz default now()
);
```

**Missing constraint** (known gap): no `UNIQUE(word_id, username)` — the frontend prevents duplicate votes but the database does not.

## Tech Stack
- Vanilla HTML/CSS/JS (single `index.html`)
- Supabase anonymous REST API (anon key embedded in frontend)
- Canvas 2D for triangle rendering
- GitHub Pages for hosting

## Known Limitations / Open Items
- No persistence of username across page reloads (intentional simplicity).
- No admin UI to delete words or ratings.
- No RLS policies configured in Supabase — relies on the secret token being obscure.
- The triangle canvas uses fixed internal dimensions (440×360 / 440×320 / 440×380) scaled via CSS to 100% width.
