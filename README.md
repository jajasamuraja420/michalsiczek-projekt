# Sudoku Web

Gra Sudoku działająca w przeglądarce: generowanie plansz w trzech poziomach trudności, podpowiedzi, notatki, licznik czasu, zapis postępu oraz ranking graczy.

## Opis projektu

**Cel:** stworzenie aplikacji webowej pozwalającej grać w Sudoku (plansza 9x9) z walidacją ruchów w czasie rzeczywistym, zapisem stanu gry i rankingiem wyników.

**Stos technologiczny**

| Warstwa | Technologia |
|---|---|
| Frontend | TypeScript, Vite, HTML5/CSS3 (SPA) |
| Backend | Node.js, Express (REST API) |
| Baza danych | PostgreSQL |
| Testy | Vitest (jednostkowe), Playwright (e2e) |
| CI | GitHub Actions |

**Funkcje**

- nowa gra: poziom łatwy / średni / trudny (generator z gwarancją jednego rozwiązania),
- wpisywanie cyfr, notatki (kandydaci), cofnij / ponów,
- podświetlanie konfliktów, podpowiedź, sprawdzanie planszy,
- licznik czasu, pauza,
- rejestracja i logowanie, zapis i wznowienie gry,
- ranking najlepszych czasów.

## Struktura repozytorium

```
sudoku-web/
├── README.md
├── docs/
│   └── DOKUMENTACJA.md      # use case, diagramy, ERD, API
├── frontend/
│   ├── src/
│   │   ├── domain/          # Board, Cell, Solver, Generator, Validator
│   │   ├── ui/              # komponenty widoku
│   │   └── api/             # klient REST
│   └── tests/
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── services/
│   │   └── repositories/
│   └── tests/
├── db/
│   └── schema.sql
└── .github/workflows/ci.yml
```

## Uruchomienie

```bash
git clone https://github.com/<user>/sudoku-web.git
cd sudoku-web

# baza danych
psql -f db/schema.sql

# backend
cd backend && npm install && npm run dev

# frontend (nowy terminal)
cd frontend && npm install && npm run dev
```

Aplikacja: http://localhost:5173, API: http://localhost:3000/api

## Dokumentacja

Pełna dokumentacja (przypadki użycia, diagramy aktywności, sekwencji, klas, ERD, API) znajduje się w [docs/DOKUMENTACJA.md](docs/DOKUMENTACJA.md).

## Konwencje pracy w repozytorium

- gałęzie: `main` (stabilna), `develop`, `feature/<nazwa>`,
- commity w stylu Conventional Commits (`feat:`, `fix:`, `docs:`),
- Pull Request wymaga zielonego CI i jednego review.
