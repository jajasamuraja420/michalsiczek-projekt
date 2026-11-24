# Dokumentacja projektu: Sudoku Web

## Spis treści

1. Wymagania
2. Przypadki użycia
3. Diagramy aktywności
4. Diagramy sekwencji
5. Diagramy klas
6. Diagram ERD
7. Specyfikacja API
8. Architektura i testy

---

## 1. Wymagania

### Funkcjonalne

| ID | Wymaganie |
|---|---|
| F1 | System generuje planszę Sudoku w trzech poziomach trudności. |
| F2 | Gracz może wpisać cyfrę 1-9 w pustym polu; pola początkowe są zablokowane. |
| F3 | System wykrywa i podświetla konflikty (powtórzenia w wierszu, kolumnie, bloku 3x3). |
| F4 | Gracz może korzystać z notatek, cofania/ponawiania i podpowiedzi. |
| F5 | System mierzy czas gry i pozwala go zatrzymać. |
| F6 | Zalogowany gracz może zapisać i wznowić grę. |
| F7 | Po poprawnym rozwiązaniu wynik trafia do rankingu. |
| F8 | Administrator może przeglądać i usuwać wyniki oraz zarządzać zbiorem plansz. |

### Niefunkcjonalne

| ID | Wymaganie |
|---|---|
| N1 | Działanie w aktualnych wersjach Chrome, Firefox, Safari, Edge. |
| N2 | Generowanie planszy poniżej 1 s. |
| N3 | Interfejs responsywny (desktop i telefon). |
| N4 | Hasła przechowywane jako hash (bcrypt), komunikacja przez HTTPS. |
| N5 | Pokrycie testami logiki domenowej minimum 80%. |

---

## 2. Przypadki użycia

### Diagram przypadków użycia

```mermaid
flowchart LR
    Gosc(["Gość"])
    Gracz(["Gracz"])
    Admin(["Administrator"])

    subgraph System["System: Sudoku Web"]
        UC1(["Rozpocznij nową grę"])
        UC2(["Wpisz cyfrę"])
        UC3(["Dodaj notatkę"])
        UC4(["Cofnij / ponów ruch"])
        UC5(["Poproś o podpowiedź"])
        UC6(["Sprawdź planszę"])
        UC7(["Zatrzymaj / wznów czas"])
        UC8(["Zarejestruj się"])
        UC9(["Zaloguj się"])
        UC10(["Zapisz grę"])
        UC11(["Wznów zapisaną grę"])
        UC12(["Zobacz ranking"])
        UC13(["Zarządzaj wynikami"])
        UC14(["Zarządzaj planszami"])
    end

    Gosc --> UC1
    Gosc --> UC2
    Gosc --> UC8
    Gosc --> UC9
    Gosc --> UC12

    Gracz --> UC1
    Gracz --> UC2
    Gracz --> UC3
    Gracz --> UC4
    Gracz --> UC5
    Gracz --> UC6
    Gracz --> UC7
    Gracz --> UC10
    Gracz --> UC11
    Gracz --> UC12

    Admin --> UC13
    Admin --> UC14
    Admin --> UC9
```

Gracz dziedziczy możliwości gościa (po zalogowaniu). Gość może grać, ale jego wyniki i postęp nie są zapisywane.

### Scenariusze (opisy tekstowe)

**UC1: Rozpocznij nową grę**

| Pole | Opis |
|---|---|
| Aktor | Gość, Gracz |
| Warunek wstępny | Aplikacja jest załadowana. |
| Scenariusz główny | 1. Użytkownik wybiera poziom trudności. 2. System pobiera/generuje planszę. 3. System wyświetla planszę i uruchamia licznik czasu. |
| Scenariusz alternatywny | 2a. Błąd serwera: system generuje planszę lokalnie w przeglądarce. |
| Warunek końcowy | Gra jest w stanie `W_TRAKCIE`. |

**UC2: Wpisz cyfrę**

| Pole | Opis |
|---|---|
| Aktor | Gość, Gracz |
| Warunek wstępny | Gra jest w trakcie, wybrane pole jest puste (nie jest polem początkowym). |
| Scenariusz główny | 1. Użytkownik zaznacza pole. 2. Wpisuje cyfrę 1-9. 3. System zapisuje ruch i sprawdza konflikty. 4. System aktualizuje widok. |
| Scenariusz alternatywny | 2a. Użytkownik wpisuje cyfrę w polu początkowym: system ignoruje akcję. 3a. Wykryto konflikt: system podświetla pola w konflikcie. 3b. Plansza kompletna i poprawna: przejście do UC "Zakończ grę" (zapis wyniku). |
| Warunek końcowy | Stan planszy zawiera nowy ruch. |

**UC10: Zapisz grę**

| Pole | Opis |
|---|---|
| Aktor | Gracz |
| Warunek wstępny | Użytkownik jest zalogowany, gra jest w trakcie. |
| Scenariusz główny | 1. Użytkownik wybiera "Zapisz". 2. System wysyła stan planszy, czas i historię ruchów do API. 3. System potwierdza zapis. |
| Scenariusz alternatywny | 2a. Brak sieci: system zachowuje stan lokalnie i ponawia próbę później. |

**UC12: Zobacz ranking**

| Pole | Opis |
|---|---|
| Aktor | Gość, Gracz |
| Scenariusz główny | 1. Użytkownik otwiera widok rankingu. 2. Wybiera poziom trudności. 3. System wyświetla 10 najlepszych czasów. |

---

## 3. Diagramy aktywności

### 3.1. Rozgrywka (od startu do zakończenia)

```mermaid
flowchart TD
    A([Start]) --> B["Wybór poziomu trudności"]
    B --> C["Wygenerowanie planszy"]
    C --> D["Wyświetlenie planszy, start czasu"]
    D --> E{"Akcja gracza"}

    E -->|Wybór pola| F["Zaznacz pole"]
    F --> E

    E -->|Wpisanie cyfry| G{"Pole edytowalne?"}
    G -->|Nie| E
    G -->|Tak| H["Zapisz ruch"]
    H --> I{"Konflikt w wierszu, kolumnie lub bloku?"}
    I -->|Tak| J["Podświetl konflikt"]
    I -->|Nie| K["Usuń podświetlenia"]
    J --> L{"Plansza pełna?"}
    K --> L
    L -->|Nie| E
    L -->|Tak| M{"Brak konfliktów?"}
    M -->|Nie| E
    M -->|Tak| N["Zatrzymaj czas"]

    E -->|Podpowiedź| P["Wstaw poprawną cyfrę z rozwiązania"]
    P --> H

    E -->|Cofnij| Q["Przywróć poprzedni stan"]
    Q --> E

    E -->|Pauza| R["Zatrzymaj czas, ukryj planszę"]
    R --> S["Wznów"]
    S --> E

    N --> O{"Gracz zalogowany?"}
    O -->|Tak| T["Zapisz wynik w rankingu"]
    O -->|Nie| U["Pokaż komunikat o zalogowaniu"]
    T --> V([Koniec])
    U --> V
```

### 3.2. Generowanie planszy

```mermaid
flowchart TD
    A([Start]) --> B["Wygeneruj pełną, poprawną planszę backtrackingiem"]
    B --> C["Wylosuj kolejność pól do usunięcia"]
    C --> D["Weź kolejne pole, zapamiętaj cyfrę, wyczyść pole"]
    D --> E{"Liczba rozwiązań równa 1?"}
    E -->|Nie| F["Przywróć cyfrę"]
    E -->|Tak| G["Zostaw pole puste"]
    F --> H{"Osiągnięto docelową liczbę wskazówek?"}
    G --> H
    H -->|Nie| D
    H -->|Tak| I["Zwróć planszę i rozwiązanie"]
    I --> J([Koniec])
```

Docelowa liczba wskazówek: łatwy ~40, średni ~32, trudny ~26.

---

## 4. Diagramy sekwencji

### 4.1. Nowa gra

```mermaid
sequenceDiagram
    actor Gracz
    participant UI as Interfejs (SPA)
    participant GC as GameController
    participant Gen as SudokuGenerator
    participant API as REST API
    participant DB as PostgreSQL

    Gracz->>UI: Wybiera poziom trudności
    UI->>GC: startNewGame(difficulty)
    GC->>API: GET /api/puzzles/random?difficulty=...
    API->>DB: SELECT puzzle
    DB-->>API: dane planszy
    API-->>GC: 200 OK (grid, difficulty)
    alt błąd serwera
        GC->>Gen: generate(difficulty)
        Gen-->>GC: Puzzle
    end
    GC->>GC: utwórz Game, uruchom Timer
    GC-->>UI: stan gry
    UI-->>Gracz: Wyświetla planszę
```

### 4.2. Wpisanie cyfry i zakończenie gry

```mermaid
sequenceDiagram
    actor Gracz
    participant UI as Interfejs (SPA)
    participant GC as GameController
    participant B as Board
    participant V as Validator
    participant API as REST API
    participant DB as PostgreSQL

    Gracz->>UI: Klik w pole, wpisanie cyfry
    UI->>GC: placNumber(row, col, value)
    GC->>B: setValue(row, col, value)
    B-->>GC: ruch zapisany
    GC->>V: findConflicts(board)
    V-->>GC: lista konfliktów
    GC-->>UI: stan planszy + konflikty
    UI-->>Gracz: Odświeżony widok

    opt plansza kompletna i bez konfliktów
        GC->>GC: timer.stop()
        GC->>API: POST /api/games/{id}/finish (czas)
        API->>DB: UPDATE games, INSERT wynik
        DB-->>API: OK
        API-->>GC: 200 OK (pozycja w rankingu)
        GC-->>UI: gra zakończona
        UI-->>Gracz: Gratulacje i pozycja w rankingu
    end
```

### 4.3. Logowanie

```mermaid
sequenceDiagram
    actor Gracz
    participant UI as Interfejs (SPA)
    participant API as REST API
    participant Auth as AuthService
    participant DB as PostgreSQL

    Gracz->>UI: Podaje e-mail i hasło
    UI->>API: POST /api/auth/login
    API->>Auth: authenticate(email, password)
    Auth->>DB: SELECT user WHERE email
    DB-->>Auth: user (password_hash)
    Auth->>Auth: bcrypt.compare()
    alt dane poprawne
        Auth-->>API: token JWT
        API-->>UI: 200 OK (token)
        UI-->>Gracz: Zalogowano
    else dane niepoprawne
        Auth-->>API: błąd
        API-->>UI: 401 Unauthorized
        UI-->>Gracz: Komunikat o błędzie
    end
```

### 4.4. Zapis i wznowienie gry

```mermaid
sequenceDiagram
    actor Gracz
    participant UI as Interfejs (SPA)
    participant API as REST API
    participant GS as GameService
    participant DB as PostgreSQL

    Gracz->>UI: Klika "Zapisz"
    UI->>API: PUT /api/games/{id} (stan, czas, ruchy) + JWT
    API->>GS: saveGame(userId, gameId, state)
    GS->>DB: UPDATE games, INSERT moves
    DB-->>GS: OK
    GS-->>API: OK
    API-->>UI: 200 OK
    UI-->>Gracz: Potwierdzenie zapisu

    Gracz->>UI: Klika "Wznów grę"
    UI->>API: GET /api/games?status=IN_PROGRESS + JWT
    API->>DB: SELECT games, moves
    DB-->>API: dane
    API-->>UI: 200 OK (stan gry)
    UI-->>Gracz: Odtworzona plansza i czas
```

---

## 5. Diagramy klas

### 5.1. Logika domenowa (frontend)

```mermaid
classDiagram
    class Cell {
        +int row
        +int col
        +int value
        +bool isGiven
        +Set~int~ notes
        +isEmpty() bool
        +setValue(int v) void
        +toggleNote(int n) void
    }

    class Board {
        -Cell[][] cells
        +getCell(int r, int c) Cell
        +setValue(int r, int c, int v) void
        +clearCell(int r, int c) void
        +isComplete() bool
        +clone() Board
        +toArray() int[][]
    }

    class Validator {
        +findConflicts(Board b) Conflict[]
        +isValidPlacement(Board b, int r, int c, int v) bool
        +isSolved(Board b) bool
    }

    class Solver {
        +solve(Board b) Board
        +countSolutions(Board b, int limit) int
    }

    class SudokuGenerator {
        +generate(Difficulty d) Puzzle
        -fillFullBoard() Board
        -removeCells(Board b, int clues) Board
    }

    class Puzzle {
        +Board initial
        +Board solution
        +Difficulty difficulty
    }

    class Difficulty {
        <<enumeration>>
        EASY
        MEDIUM
        HARD
    }

    class Timer {
        -int elapsedSeconds
        +start() void
        +pause() void
        +resume() void
        +stop() int
    }

    class Move {
        +int row
        +int col
        +int oldValue
        +int newValue
        +Date timestamp
    }

    class MoveHistory {
        -Move[] undoStack
        -Move[] redoStack
        +push(Move m) void
        +undo() Move
        +redo() Move
    }

    class Game {
        +string id
        +Puzzle puzzle
        +Board board
        +GameStatus status
        +start() void
        +placeNumber(int r, int c, int v) void
        +useHint() void
        +undo() void
        +redo() void
        +finish() int
    }

    class GameStatus {
        <<enumeration>>
        NOT_STARTED
        IN_PROGRESS
        PAUSED
        COMPLETED
    }

    class GameController {
        +startNewGame(Difficulty d) Game
        +placeNumber(int r, int c, int v) void
        +saveGame() void
        +loadGame(string id) Game
    }

    Board "1" *-- "81" Cell
    Puzzle "1" o-- "2" Board
    Puzzle --> Difficulty
    Game "1" o-- "1" Puzzle
    Game "1" *-- "1" Board
    Game "1" *-- "1" Timer
    Game "1" *-- "1" MoveHistory
    Game --> GameStatus
    MoveHistory "1" o-- "*" Move
    Validator ..> Board
    Solver ..> Board
    SudokuGenerator ..> Solver
    SudokuGenerator ..> Puzzle
    GameController --> Game
    GameController ..> Validator
    GameController ..> SudokuGenerator
```

### 5.2. Warstwa backendu

```mermaid
classDiagram
    class AuthController {
        +register(req, res) void
        +login(req, res) void
    }
    class GameController {
        +create(req, res) void
        +update(req, res) void
        +finish(req, res) void
        +listByUser(req, res) void
    }
    class LeaderboardController {
        +top(req, res) void
    }
    class AuthService {
        +register(email, nick, password) User
        +authenticate(email, password) string
    }
    class GameService {
        +saveGame(userId, gameId, state) void
        +finishGame(userId, gameId, seconds) int
    }
    class LeaderboardService {
        +getTop(difficulty, limit) Score[]
    }
    class UserRepository {
        +findByEmail(email) User
        +save(user) void
    }
    class GameRepository {
        +findById(id) GameEntity
        +save(game) void
    }
    class PuzzleRepository {
        +findRandom(difficulty) PuzzleEntity
    }
    class User {
        +int id
        +string email
        +string nickname
        +string passwordHash
        +Role role
    }

    AuthController --> AuthService
    GameController --> GameService
    LeaderboardController --> LeaderboardService
    AuthService --> UserRepository
    GameService --> GameRepository
    GameService --> PuzzleRepository
    LeaderboardService --> GameRepository
    UserRepository ..> User
```

---

## 6. Diagram ERD

```mermaid
erDiagram
    USERS ||--o{ GAMES : "rozgrywa"
    PUZZLES ||--o{ GAMES : "jest podstawą"
    GAMES ||--o{ MOVES : "zawiera"

    USERS {
        int id PK
        varchar email UK
        varchar nickname UK
        varchar password_hash
        varchar role "PLAYER lub ADMIN"
        timestamp created_at
    }

    PUZZLES {
        int id PK
        char initial_grid "81 znaków, 0 = puste"
        char solution_grid "81 znaków"
        varchar difficulty "EASY, MEDIUM, HARD"
        timestamp created_at
    }

    GAMES {
        int id PK
        int user_id FK "NULL dla gościa"
        int puzzle_id FK
        char current_grid "81 znaków"
        varchar notes_json "notatki kandydatów"
        varchar status "IN_PROGRESS, PAUSED, COMPLETED"
        int elapsed_seconds
        int hints_used
        timestamp started_at
        timestamp finished_at
    }

    MOVES {
        int id PK
        int game_id FK
        int row_idx
        int col_idx
        int old_value
        int new_value
        int move_number
        timestamp created_at
    }
```

### Schemat SQL (`db/schema.sql`)

```sql
CREATE TABLE users (
    id            SERIAL PRIMARY KEY,
    email         VARCHAR(255) NOT NULL UNIQUE,
    nickname      VARCHAR(50)  NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role          VARCHAR(10)  NOT NULL DEFAULT 'PLAYER',
    created_at    TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE TABLE puzzles (
    id            SERIAL PRIMARY KEY,
    initial_grid  CHAR(81)    NOT NULL,
    solution_grid CHAR(81)    NOT NULL,
    difficulty    VARCHAR(10) NOT NULL CHECK (difficulty IN ('EASY','MEDIUM','HARD')),
    created_at    TIMESTAMP   NOT NULL DEFAULT NOW()
);

CREATE TABLE games (
    id              SERIAL PRIMARY KEY,
    user_id         INT REFERENCES users(id) ON DELETE CASCADE,
    puzzle_id       INT NOT NULL REFERENCES puzzles(id),
    current_grid    CHAR(81) NOT NULL,
    notes_json      TEXT,
    status          VARCHAR(12) NOT NULL DEFAULT 'IN_PROGRESS',
    elapsed_seconds INT NOT NULL DEFAULT 0,
    hints_used      INT NOT NULL DEFAULT 0,
    started_at      TIMESTAMP NOT NULL DEFAULT NOW(),
    finished_at     TIMESTAMP
);

CREATE TABLE moves (
    id          SERIAL PRIMARY KEY,
    game_id     INT NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    row_idx     SMALLINT NOT NULL CHECK (row_idx BETWEEN 0 AND 8),
    col_idx     SMALLINT NOT NULL CHECK (col_idx BETWEEN 0 AND 8),
    old_value   SMALLINT NOT NULL DEFAULT 0,
    new_value   SMALLINT NOT NULL DEFAULT 0,
    move_number INT NOT NULL,
    created_at  TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_games_user ON games(user_id);
CREATE INDEX idx_games_ranking ON games(status, elapsed_seconds);
CREATE INDEX idx_moves_game ON moves(game_id);
```

Ranking nie ma osobnej tabeli. Jest widokiem na `games` ze statusem `COMPLETED`, posortowanym po `elapsed_seconds` (osobno dla każdego poziomu trudności).

---

## 7. Specyfikacja API

| Metoda | Endpoint | Opis | Auth |
|---|---|---|---|
| POST | `/api/auth/register` | Rejestracja | nie |
| POST | `/api/auth/login` | Logowanie, zwraca JWT | nie |
| GET | `/api/puzzles/random?difficulty=EASY` | Losowa plansza | nie |
| POST | `/api/games` | Utworzenie gry | tak |
| GET | `/api/games?status=IN_PROGRESS` | Gry użytkownika | tak |
| PUT | `/api/games/{id}` | Zapis stanu gry | tak |
| POST | `/api/games/{id}/finish` | Zakończenie, wynik do rankingu | tak |
| GET | `/api/leaderboard?difficulty=HARD&limit=10` | Ranking | nie |
| DELETE | `/api/admin/games/{id}` | Usunięcie wyniku | admin |

Przykład odpowiedzi `GET /api/puzzles/random?difficulty=EASY`:

```json
{
  "id": 17,
  "difficulty": "EASY",
  "grid": "530070000600195000098000060800060003400803001700020006060000280000419005000080079"
}
```

---

## 8. Architektura i testy

```mermaid
flowchart LR
    P["Przeglądarka: SPA TypeScript"] -->|HTTPS, JSON| A["Backend: Node.js + Express"]
    A --> D[("PostgreSQL")]
```

**Strategia testów**

| Poziom | Zakres | Narzędzie |
|---|---|---|
| Jednostkowe | Validator, Solver, Generator, MoveHistory | Vitest |
| Integracyjne | Endpointy API z testową bazą | Vitest + Supertest |
| End-to-end | Rozegranie gry, logowanie, zapis | Playwright |

**CI (GitHub Actions):** instalacja, lint, testy jednostkowe i integracyjne, build frontendu przy każdym Pull Requeście.

**Rozwój w przyszłości:** tryb dzienny (ta sama plansza dla wszystkich), warianty (Killer, Samurai), tryb offline (PWA).
