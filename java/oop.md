# Programowanie obiektowe (OOP) – kompozycja vs dziedziczenie

W programowaniu obiektowym (**OOP – Object-Oriented Programming**) jednym z kluczowych zagadnień projektowych jest wybór między **dziedziczeniem** a **kompozycją** jako mechanizmem ponownego użycia kodu i budowania relacji między klasami.  

Na poziomie **mid developera** oczekuje się nie tylko znajomości definicji, ale przede wszystkim umiejętności świadomego podejmowania decyzji projektowych wraz ze zrozumieniem konsekwencji architektonicznych.

---

## 1. Dziedziczenie (Inheritance)

### Definicja

Dziedziczenie to mechanizm, w którym klasa potomna (*subclass*) przejmuje pola i metody klasy bazowej (*superclass*). Relacja ta wyraża zależność typu:

> **is-a** („jest rodzajem”)

Przykłady:
- `Dog` **jest** `Animal`
- `Manager` **jest** `Employee`

### Kluczowe właściwości

1. **Polimorfizm** – możliwość traktowania obiektu klasy potomnej jako obiektu klasy bazowej.
2. **Współdzielenie implementacji** – reużycie kodu z klasy bazowej.
3. **Nadpisywanie metod (override)** – zmiana zachowania metody bazowej.
4. **Zasada LSP (Liskov Substitution Principle)** – obiekt klasy potomnej musi móc zastąpić obiekt klasy bazowej bez zmiany poprawności programu.

### Zalety

- Naturalne modelowanie hierarchii domenowych.
- Silne wsparcie dla polimorfizmu.
- Czytelność przy dobrze zaprojektowanej abstrakcji.

### Wady

- Silne sprzężenie (*tight coupling*).
- Zmiany w klasie bazowej wpływają na wszystkie klasy potomne.
- Problem **fragile base class**.
- Brak elastyczności w czasie działania (brak możliwości dynamicznej zmiany typu).

---

## 2. Kompozycja (Composition)

### Definicja

Kompozycja polega na budowaniu klasy poprzez użycie innych obiektów jako jej pól. Relacja:

> **has-a** („ma”)

Przykłady:
- `Car` ma `Engine`
- `Order` ma `PaymentStrategy`

### Właściwości

1. Delegowanie odpowiedzialności do obiektów składowych.
2. Mniejsze sprzężenie między klasami.
3. Możliwość dynamicznej zmiany zachowania (np. wzorzec Strategy).

### Zalety

- Większa elastyczność.
- Lepsza zgodność z zasadami **SOLID**:
  - **SRP** – Single Responsibility Principle
  - **OCP** – Open/Closed Principle
- Lepsza testowalność (Dependency Injection).
- Ograniczenie problemu głębokich hierarchii.

### Wady

- Więcej konfiguracji obiektów.
- Konieczność jawnej delegacji metod.
- Może być mniej intuicyjna przy prostych relacjach typów.

---

## 3. Porównanie projektowe

| Kryterium | Dziedziczenie | Kompozycja |
|------------|---------------|------------|
| Relacja | is-a | has-a |
| Sprzężenie | Silne | Luźniejsze |
| Elastyczność | Niska | Wysoka |
| Runtime swap | Nie | Tak |
| Ryzyko naruszenia LSP | Wysokie | Niskie |
| Rozszerzalność | Hierarchia | Delegacja / dekorowanie |

---

## 4. „Prefer composition over inheritance”

Zasada spopularyzowana przez książkę  
*Design Patterns: Elements of Reusable Object-Oriented Software* (Gang of Four, 1994).

Nie oznacza ona, że dziedziczenie jest złe. Oznacza, że:

- dziedziczenie powinno modelować **relację typu**
- kompozycja powinna służyć do **ponownego użycia zachowania**

---

## 5. Typowe błędy projektowe

### ❌ Nadużywanie dziedziczenia

Problem:
Bird

├── FlyingBird

└── Ostrich


Jeśli `Bird` posiada metodę `fly()`, a `Ostrich` nie lata – naruszamy LSP.

### ❌ Dziedziczenie wyłącznie dla reużycia kodu

Tworzenie sztucznej klasy bazowej tylko po to, aby uniknąć duplikacji, prowadzi do złej abstrakcji.

Lepsze rozwiązanie:
- wyodrębnić wspólne zachowanie do osobnej klasy
- użyć kompozycji

### ❌ Głębokie hierarchie

Hierarchie 5–7 poziomowe są trudne w utrzymaniu i testowaniu.

---

## 6. Kiedy dziedziczenie jest dobrym wyborem?

✔ Stabilna relacja typu  
✔ Dobrze zdefiniowana abstrakcja  
✔ Wymuszenie kontraktu przez klasę abstrakcyjną  
✔ Frameworki wymagające dziedziczenia  

---

## 7. Kiedy kompozycja jest lepsza?

✔ Dynamiczna zmiana zachowania  
✔ Unikanie sztywnej hierarchii  
✔ Implementacja wzorców:
- Strategy  
- Decorator  
- Bridge  
- Composite  

✔ Dependency Injection  

---

## 8. Wyjątki od reguły

1. Frameworki wymagające dziedziczenia.
2. Silna taksonomia domenowa.
3. Optymalizacje wydajnościowe.
4. sealed/final classes ograniczające ryzyko.

---

## 9. Pytania rekrutacyjne (mid-level) + przykładowe odpowiedzi

### 1. Czym różni się relacja „is-a” od „has-a”?

**Odpowiedź (mid-level):**  
Relacja *is-a* oznacza, że klasa potomna jest wyspecjalizowaną wersją klasy bazowej i musi spełniać jej kontrakt — zgodnie z LSP. Relacja *has-a* oznacza, że klasa wykorzystuje inny obiekt jako część swojej implementacji. W praktyce używam dziedziczenia, gdy modeluję stabilną hierarchię typów, natomiast kompozycji, gdy chcę budować zachowanie i ograniczać sprzężenie.



### 2. Podaj przykład naruszenia LSP.

**Odpowiedź:**  
Klasyczny przykład to `Rectangle` i `Square`, gdzie `Square` dziedziczy po `Rectangle`, ale zmiana szerokości wpływa na wysokość. W takim przypadku obiekt `Square` nie może być bezpiecznie używany tam, gdzie oczekuje się `Rectangle`, bo zmienia semantykę operacji. To oznacza, że hierarchia została zaprojektowana błędnie.


### 3. Dlaczego kompozycja zmniejsza sprzężenie?

**Odpowiedź:**  
Ponieważ zależność jest realizowana przez interfejs lub abstrakcję, a nie przez twardą relację dziedziczenia. Mogę podmienić implementację w czasie działania lub w testach. Klasa nie jest zależna od szczegółów implementacyjnych klasy bazowej.


### 4. Czy dziedziczenie zawsze łamie OCP?

**Odpowiedź:**  
Nie. Dziedziczenie może wspierać OCP, jeśli rozszerzamy zachowanie przez nowe klasy potomne bez modyfikacji klasy bazowej. Problem pojawia się wtedy, gdy klasa bazowa musi być modyfikowana przy każdym nowym przypadku.


### 5. Czy interfejs to forma dziedziczenia?

**Odpowiedź:**  
Tak, to dziedziczenie kontraktu, ale bez współdzielenia implementacji. Umożliwia polimorfizm przy minimalnym sprzężeniu.


### 6. Czy można łączyć oba podejścia?

**Odpowiedź:**  
Tak. Często używam dziedziczenia do modelowania typu (np. wspólna klasa bazowa), a kompozycji do budowania zachowania wewnątrz klasy. To podejście daje najlepszą równowagę między czytelnością a elastycznością.


---

## 10. Jak odpowiadać na rozmowie

1. Zdefiniuj oba mechanizmy.
2. Wyjaśnij is-a vs has-a.
3. Omów LSP i sprzężenie.
4. Wspomnij zasadę „prefer composition over inheritance”.
5. Podaj przykład praktyczny.
6. Wskaż wyjątki.

### Przykładowa odpowiedź (gotowiec do powiedzenia)

„Dziedziczenie wykorzystuję wtedy, gdy istnieje realna relacja typu *is-a* i jestem pewien stabilności abstrakcji. Problem z dziedziczeniem polega na silnym sprzężeniu i ryzyku naruszenia LSP.  

Dlatego w większości przypadków preferuję kompozycję, szczególnie gdy buduję zachowanie lub chcę umożliwić jego dynamiczną zmianę. Kompozycja lepiej wspiera OCP i testowalność, bo mogę podmieniać zależności przez interfejsy.  

Jednocześnie nie traktuję tej zasady dogmatycznie — w stabilnych hierarchiach domenowych dziedziczenie jest w pełni uzasadnione.”


---

## 11. Dojrzała perspektywa mid developera

Na poziomie mid chodzi o:

- świadome zarządzanie zależnościami
- analizę kosztu zmian
- przewidywanie rozwoju systemu
- unikanie nadmiernej abstrakcji

Najczęstszy błąd juniorów: projektowanie pod hipotetyczne rozszerzenia.  
Najczęstszy błąd seniorów: nadmierna komplikacja architektury.