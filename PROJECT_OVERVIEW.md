# BandPath — Project Overview

_Last updated: 2026-07-07_

## 1. What BandPath Is

BandPath is a **Telegram Mini App** for IELTS Academic exam preparation. It's a static
website (no backend/server, no database) hosted on **GitHub Pages**, opened inside
Telegram as a WebApp. Users practice full-length **Reading** and **Listening** mock
tests with the same timing, format, and scoring conventions as the real IELTS exam.

**Repo:** `https://github.com/CProprojects/BandPath`
**Bot/entry point:** `https://t.me/mockieltstest` (linked from results screen)

## 2. What the User Wants From This App

- A free, self-serve IELTS practice tool distributed via Telegram (low friction — no
  install, no signup).
- Faithful reproduction of real IELTS test structure: 3 reading passages / 40 questions
  in 60 minutes; 4 listening sections / 40 questions in ~30 minutes, audio playable once.
- Growing library of **real/authentic past IELTS papers** (sourced from PDFs the user
  supplies, with answer keys), not synthetic content — faithfulness to the source
  material is a hard requirement, including quirks/typos in the original if the answer
  key depends on them.
- Clean, consistent visual identity across every test (dark/light theme, same palette,
  same UX chrome: timer, question palette, flag/notes, review mode with correct-answer
  reveal, band-score estimate).
- Zero-maintenance-cost infrastructure: local backups and a change log that don't
  burn API/token budget to maintain.
- Fast iteration: supply a PDF → get a fully working test added to the app and pushed
  live with no manual steps on the user's part beyond dropping in audio files.

## 3. Current State (as of last update)

### 3.1 Architecture
- `index.html` — dashboard/home screen. Lists all tests via a `testsData` array
  (id, type, name, file, difficulty, question count, time limit). Filters by type
  (reading/listening) and difficulty.
- Each test is a **fully self-contained HTML file** (inline `<style>` + `<script>`,
  no shared JS/CSS files) — e.g. `bandpath_reading_test8.html`,
  `bandpath_listening_test3.html`. This means every test file duplicates the whole
  engine; changes to shared behavior (e.g. a bug fix) must be applied file-by-file.
- Telegram WebApp SDK wired into every page: `tg.ready()`, `tg.expand()`,
  `enableClosingConfirmation()`, `disableVerticalSwipes()`, `requestFullscreen()` —
  each wrapped in its own `try/catch` (a past bug: an uncaught throw here silently
  broke the whole page — see §5).
- Answers/flags/timer/notes persist in `localStorage` per test
  (`ielts_t{N}_*` for reading, `ielts_ls{N}_*` for listening — each test has a unique
  key prefix to avoid cross-test collisions).

### 3.2 Reading engine
Question types supported: `fill` (text input — covers note-completion, TFNG-adjacent
fill blanks, matching-by-letter, paragraph lookup, word-bank summaries), `tfng`
(TRUE/FALSE/NOT GIVEN), `yesno` (YES/NO/NOT GIVEN), `mcq` (radio), `multi2` (checkbox
pair for "choose TWO letters" questions, contributes 2 marks). Generic bank renderers
(`renderPersonBank`, `renderWordBank`) are reused for people/company/heading/sentence-
ending matching banks.

### 3.3 Listening engine
Left-hand audio panel with one `<audio>` player per section (plays **once only**,
enforced in the UI), right-hand scrollable question panel, section tabs (Part 1–4),
bottom question palette + submit bar — mirrors the real IELTS computer-delivered test
layout. Question types: `fill` (notes/table/map-labelling/matching-by-letter), `mcq`.
Map/floor-plan labelling questions (where the source PDF has an image) are
approximated with a simplified CSS-grid schematic reconstructing the relative layout,
since there's no way to embed the original scanned image.

### 3.4 Shared UX
- Timer with warning/critical color states, auto-submits at zero.
- Flaggable questions, scratch notes panel, draggable splitter (listening) between
  audio and questions panels.
- Results screen: band score (raw-score → band conversion table), per-section
  breakdown, full review mode with correct-answer reveal.
- Settings panel: theme (light/dark), font size, Restart Test, **Home** button (added
  to every test after initially missing).

### 3.5 Content inventory

**Reading (11 tests, 40 Q / 60 min each):**
1. Sweet Trouble · 2. Prosopagnosia · 3. Thames Tunnel · 4. Office Lighting ·
5. Why Study History · 6. Humans and Food · 7. Listening to Ocean ·
8. What Lucy Taught Us · 9. The Wonder Plant · 10. Otter ·
11. Trans-Atlantic Cable

**Listening (7 tests, 40 Q / ~30 min each, audio hosted as mp3 in repo root):**
1. Apartments & Travel (Arillas / Skiing / Nursing Program / Dinner & Environment)
2. Cleaning Agency & Museum (Homecare Cleaning / Museum Tour / Business Analysis / Employment Survey)
3. Photo Reprint & Telescope (Newspaper Photo / Hospitality Training / Battery-Powered Motorbikes / Telescope)
4. Fun Run & Textile Design (Holiday Rental / Bridge to Brisbane / Farmers' Attitudes / Aboriginal Textile Design)
5. Curling & Da Vinci (Birthday Party / The Game of Curling / Scientific Art Investigation / Sustainability)
6. Insurance & Turtles (Travel Insurance / Eyesaver Charity / Rangi's Degree Discussion / Leatherback Turtles)
7. Railway Station & Crocodiles (International Club / Fitchton Railway Station / Volcano Presentation / Crocodile Migration)

Audio naming convention: `listening_test{N}_s{1-4}.mp3`, one file per section, dropped
in the repo root (same folder as the HTML).

### 3.6 Infrastructure
- **Git/GitHub:** all changes auto-committed and pushed with no confirmation prompt
  (standing user instruction). GitHub Pages serves the live site directly from `main`.
- **Local daily backup:** `D:\BandPath Backup\backup_script.ps1`, run daily at 23:50
  via Windows Task Scheduler ("BandPath Daily Backup"), zips the whole `e:\BandPath`
  (excluding `.git`) to `BandPath_DD.MM.YYYY(N).zip` — `N` is a counter that never
  resets and never gets pruned (user has 100GB+ free, wants every backup kept forever).
- **Change log:** Google Sheet "BandPath Change Log" (via Google Drive MCP connector),
  columns `ID; Дата; Тип; Изменение`, continuously incrementing ID (id001, id002, …,
  never resets daily), emoji-tagged type column (🔧 Настройка / 🐛 Багфикс / ✨ Фича /
  📘 Новый тест). Two malformed early duplicate versions of this sheet still exist in
  the user's Drive and need manual deletion (no delete capability via the available
  Drive MCP tools).

## 4. Known Gaps / Incomplete Items

- **Reading Test 5** (`bandpath_reading_test5.html`): `THEORIES_P1` word bank is
  missing letters A and E; `WORDS_P1` word bank is missing A, C, F, G, J. Reconstructed
  from partial context only — source PDF for the missing letters was never supplied.
- **Reading Test 6** (`bandpath_reading_test6.html`): `WORDS_P3` word bank is missing
  5 of 10 letters (A, E, F, G, H) for the same reason.
- Both are flagged in-app only implicitly (missing bank entries) — no visible warning
  to the end user.
- Listening map/floor-plan questions use simplified reconstructed schematics rather
  than the original scanned images (visually approximate, not pixel-accurate).

## 5. Notable Past Bugs (fixed)

- **Infinite "Loading…" on the live site:** an uncaught throw from
  `tg.requestFullscreen()` (called outside a real Telegram client / without a user
  gesture) aborted the top-level `<script>` before `window.onload` ever registered.
  Fixed by wrapping every Telegram API call in its own `try/catch` across all files.
- **localStorage key collisions:** Reading Test 7 was originally using Test 8's key
  prefix (`ielts_t8_*`), corrupting/sharing progress between the two tests.
- Several reading tests called undefined bank-render functions for paragraph-matching
  questions that don't need a bank (dead code from copy-paste template drift).
- `multi2` ("choose TWO letters") question type was silently unhandled in early
  reading tests — added full support (checkbox pair, weighted 2-point scoring, review-
  mode markup, palette flashing across both question numbers).

## 6. Future Plans / Open Items

- Continue adding Listening tests (8, 9, … ) from user-supplied PDFs, following the
  established pipeline: check for topic duplicates → verify answer key → build from
  the most recently verified test as a template → wire up `testsData` → commit + push.
- Same ongoing pipeline for further Reading tests beyond 11.
- Optionally backfill the missing word-bank letters in Reading Tests 5 and 6 if the
  user ever supplies the original source PDFs.
- User to manually delete the 2 stale "BandPath Change Log" sheet copies from Google
  Drive.
- No current plans (as of this writing) to move off the "one fully self-contained
  HTML file per test" architecture, despite the code-duplication cost — this has been
  the accepted tradeoff for simplicity and zero build-step deployment.
