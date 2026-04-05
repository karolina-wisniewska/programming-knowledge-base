# Kolekcje w Java – baza wiedzy (mid-level)

## 1. Wprowadzenie

Framework kolekcji w Javie (Java Collections Framework, JCF) to zestaw interfejsów i klas służących do przechowywania oraz manipulowania grupami obiektów. Znajduje się głównie w pakiecie `java.util`.

Główne cele:
- ujednolicenie operacji na kolekcjach,
- zapewnienie wydajnych implementacji struktur danych,
- umożliwienie stosowania algorytmów (np. sortowanie, wyszukiwanie),
- wspieranie programowania generycznego (Generics).

---

## 2. Hierarchia kolekcji

### 2.1 Główne interfejsy

```text
Iterable
└── Collection
    ├── List
    ├── Set
    ├── Queue
    └── Deque

Map (osobna hierarchia)
```

### 2.2 Charakterystyka

| Interfejs | Opis |
|----------|------|
| `Collection` | Bazowy interfejs dla większości kolekcji |
| `List` | Kolekcja uporządkowana, dopuszcza duplikaty |
| `Set` | Brak duplikatów |
| `Queue` | Kolejka (FIFO lub inne strategie) |
| `Deque` | Dwukierunkowa kolejka |
| `Map` | Struktura klucz-wartość |

---

## 3. Listy (List)

### 3.1 Implementacje

#### ArrayList
- dynamiczna tablica
- szybki dostęp przez indeks: **O(1)**
- dodawanie na koniec: amortyzowane **O(1)**
- wstawianie/usuwanie w środku: **O(n)**

#### LinkedList
- lista dwukierunkowa
- dostęp przez indeks: **O(n)**
- dodawanie/usuwanie na początku/końcu: **O(1)**

#### Vector (legacy)
- synchronizowany (thread-safe)
- rzadko używany

---

### 3.2 Kiedy używać?

| Sytuacja | Wybór |
|--------|------|
| częsty dostęp po indeksie | ArrayList |
| częste wstawianie/usuwanie w środku | LinkedList |
| wielowątkowość (raczej nie) | Collections.synchronizedList |

---

## 4. Set

### 4.1 Implementacje

#### HashSet
- brak kolejności
- oparty na HashMap
- operacje: **O(1)** (średnio)

#### LinkedHashSet
- zachowuje kolejność wstawiania

#### TreeSet
- sortowany (Red-Black Tree)
- operacje: **O(log n)**

---

### 4.2 Wymagania

Dla HashSet:
- poprawna implementacja `equals()` i `hashCode()`

---

## 5. Map

### 5.1 Implementacje

#### HashMap
- brak kolejności
- klucz może być `null` (1 raz)
- operacje: **O(1)**

#### LinkedHashMap
- zachowuje kolejność (insertion lub access)

#### TreeMap
- sortowana mapa
- operacje: **O(log n)**

#### ConcurrentHashMap
- bezpieczna wątkowo
- brak globalnej blokady

---

### 5.2 HashMap – szczegóły

- oparta na tablicy bucketów
- kolizje rozwiązywane przez:
  - listy (do Java 8)
  - drzewa czerwono-czarne (od Java 8 przy dużych kolizjach)

---

## 6. Queue i Deque

### Queue
- FIFO
- metody:
  - `offer()` – dodanie
  - `poll()` – usunięcie
  - `peek()` – podgląd

### Deque
- operacje na obu końcach
- może działać jako:
  - stos (LIFO)
  - kolejka (FIFO)

Implementacje:
- `ArrayDeque` (najczęściej używana)
- `LinkedList`

---

## 7. Złożoność obliczeniowa (Big-O)

| Operacja | ArrayList | LinkedList | HashSet | TreeSet | HashMap |
|--------|----------|-----------|--------|--------|--------|
| get | O(1) | O(n) | - | - | O(1) |
| add | O(1) | O(1) | O(1) | O(log n) | O(1) |
| remove | O(n) | O(1)* | O(1) | O(log n) | O(1) |

\* dla znanego elementu (iterator)

---

## 8. Iteracja po kolekcjach

### 8.1 Iterator

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String val = it.next();
}
```

### 8.2 Enhanced for (for-each)

```java
for (String s : list) {
    System.out.println(s);
}
```
#### Charakterystyka

- działa na obiektach implementujących Iterable
- upraszcza składnię względem użycia Iterator
- nie pozwala na bezpieczne usuwanie elementów w trakcie iteracji

#### Jak to działa wewnętrznie

kompilator zamienia tę konstrukcję na użycie Iterator
### 8.3 Stream API
```java
list.stream()
    .filter(s -> s.length() > 3)
    .forEach(System.out::println);
```

#### Cechy

- podejście deklaratywne (opisujemy co, a nie jak)
- operacje:
  - pośrednie: filter, map, sorted
  - terminalne: forEach, collect, reduce
- możliwość przetwarzania równoległego: parallelStream()

## 9. Fail-fast vs Fail-safe

### 9.1 Fail-fast

wykrywa modyfikację kolekcji podczas iteracji

rzuca ConcurrentModificationException

dotyczy większości standardowych kolekcji (ArrayList, HashMap)

#### Przykład

```java
for (String s : list) {
    list.remove(s); // wyjątek
}
```

### 9.2 Fail-safe

operuje na kopii kolekcji

nie rzuca wyjątku

zmiany nie są widoczne w trakcie iteracji

#### Przykłady

- ConcurrentHashMap
- CopyOnWriteArrayList
## 10. Synchronizacja

### 10.1 Collections.synchronized

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

#### Cechy

- opakowuje kolekcję w synchronizowany wrapper
- blokuje cały obiekt (co ogranicza skalowalność)
### 10.2 Kolekcje współbieżne

- ConcurrentHashMap
- CopyOnWriteArrayList

#### Zalety

- lepsza wydajność w środowisku wielowątkowym
- brak globalnej blokady
### 10.3 CopyOnWriteArrayList

#### Zasada działania

każda operacja modyfikująca tworzy nową kopię tablicy

#### Zastosowanie

scenariusze z dużą liczbą odczytów i małą liczbą zapisów

#### Wady

wysoki koszt pamięciowy i czasowy przy zapisie
## 11. equals() i hashCode()

### 11.1 Kontrakt
- jeśli equals() zwraca true, to hashCode() musi być taki sam
- jeśli hashCode() jest taki sam, equals() nie musi zwracać true
### 11.2 Problem praktyczny
```java
Set<Person> set = new HashSet<>();
set.add(new Person("Jan"));

System.out.println(set.contains(new Person("Jan"))); // false
```

#### Przyczyna

brak implementacji equals() i hashCode()
### 11.3 Skutki błędów
- duplikaty w Set
- brak możliwości odnalezienia elementów w Map
## 12. Comparable vs Comparator

### 12.1 Comparable
definiuje naturalny porządek
implementowany w klasie
```java
class Person implements Comparable<Person> {
    int age;

    public int compareTo(Person o) {
        return Integer.compare(this.age, o.age);
    }
}
```
### 12.2 Comparator

definiuje alternatywne sposoby sortowania
implementowany poza klasą
```java
Comparator<Person> byName = (a, b) -> a.name.compareTo(b.name);
```
### 12.3 Różnice
| Cecha | Comparable | Comparator |
|--------|------------|------------|
| implementacja | w klasie | poza klasą |
| liczba strategii | jedna | wiele |
| zastosowanie | naturalne sortowanie | różne sposoby sortowania |
## 13. Najczęstsze błędy
- brak implementacji hashCode() przy użyciu HashSet lub HashMap
- używanie LinkedList bez uzasadnienia zamiast ArrayList
- ignorowanie złożoności obliczeniowej operacji
- modyfikowanie kolekcji w trakcie iteracji
- stosowanie synchronizacji zamiast kolekcji współbieżnych
- zmiana stanu obiektu używanego jako klucz w Map
## 14. Przykładowe pytania rekrutacyjne (mid)
1. Dlaczego nie można usuwać elementów w pętli for-each?

Odpowiedź:
Ponieważ for-each używa iteratora typu fail-fast. Bezpośrednia modyfikacja kolekcji powoduje ConcurrentModificationException.

2. Jak bezpiecznie usuwać elementy podczas iteracji?

Odpowiedź:
Należy użyć iteratora i metody remove():

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().length() < 3) {
        it.remove();
    }
}
```
3. Kiedy użyć Stream API zamiast pętli?

Odpowiedź:
Gdy operacje są transformacyjne (filter, map, grouping) i chcemy pisać kod bardziej deklaratywnie.

4. Różnica między fail-fast a fail-safe?

Odpowiedź:
Fail-fast rzuca wyjątek przy modyfikacji kolekcji, fail-safe działa na kopii i nie zgłasza błędu.

5. Dlaczego ConcurrentHashMap jest bardziej wydajna niż synchronizedMap?

Odpowiedź:
Ponieważ stosuje drobnoziarnistą synchronizację zamiast blokować całą strukturę.

6. Co się stanie, jeśli hashCode nie jest zgodny z equals?

Odpowiedź:
Struktury oparte o hashowanie będą działać niepoprawnie – elementy mogą być nieosiągalne.

7. Kiedy użyć CopyOnWriteArrayList?

Odpowiedź:
Gdy liczba odczytów znacząco przewyższa liczbę zapisów.

8. Czy Stream API zawsze jest szybsze?

Odpowiedź:
Nie. Może być wolniejsze przez narzut. Zyski są widoczne przy operacjach równoległych lub złożonych pipeline’ach.

9. Czym różni się Comparator od Comparable w praktyce?

Odpowiedź:
Comparable definiuje jedną naturalną kolejność, Comparator pozwala na wiele różnych strategii sortowania.

10. Dlaczego zmiana pola używanego w hashCode jest problemem?

Odpowiedź:
Obiekt może trafić do innego bucketu i nie będzie możliwy do odnalezienia.

## 15. Podsumowanie

Najważniejsze umiejętności:

rozumienie działania iteracji i iteratorów
znajomość różnicy fail-fast vs fail-safe
świadome użycie kolekcji współbieżnych
poprawna implementacja equals() i hashCode()
wybór odpowiedniej strategii sortowania
efektywne użycie Stream API
świadomość kosztów operacji (czasowych i pamięciowych)