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

### 5.3 Zależność między HashSet a HashMap

#### 1. Kluczowa idea

`HashSet` jest w rzeczywistości **oparty na `HashMap`**.

Oznacza to, że:
- `HashSet` **nie przechowuje elementów samodzielnie**
- wewnętrznie używa `HashMap`, gdzie:
  - element zbioru (`Set`) jest kluczem (`key`)
  - wartość (`value`) jest stałą atrapą (dummy value)

---

#### 2. Implementacja wewnętrzna

Uproszczona wersja implementacji `HashSet`:

```java
public class HashSet<E> extends AbstractSet<E> {
    private transient HashMap<E, Object> map;

    private static final Object PRESENT = new Object();

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```
---

#### 3. Co to oznacza w praktyce?
3.1 Brak duplikatów
- `HashMap` nie pozwala na duplikaty kluczy
- `HashSet` wykorzystuje tę właściwość

```java
set.add("A");
set.add("A"); // drugi element zostanie zignorowany
```
Dlaczego?

- `map.put(key, value)` nadpisuje istniejący wpis
- `HashSet` interpretuje to jako brak dodania nowego elementu

---

3.2 equals() i hashCode()

Zachowanie `HashSet` zależy bezpośrednio od implementacji `HashMap`, czyli:

- najpierw używany jest `hashCode()`
- potem `equals()`

Proces:

1. obliczenie `hashCode()`
2. znalezienie bucketu
3. porównanie przez `equals()`

---

3.3 Złożoność operacji

Ponieważ `HashSet` używa `HashMap`:

| Operacja | Złożoność |
| -------- | --------- |
| add      | O(1)      |
| remove   | O(1)      |
| contains | O(1)      |

(w średnim przypadku)

---

#### 4. Jak wygląda struktura w pamięci?
HashMap:

```bucket[] -> (key, value)```

HashSet:

```bucket[] -> (element, PRESENT)```

Czyli:

HashSet to de facto **HashMap bez znaczących wartości**

---

#### 5. Konsekwencje tej zależności
5.1 Wydajność

identyczna jak `HashMap` dla operacji na kluczach

---

5.2 Zużycie pamięci

`HashSet` zużywa więcej pamięci niż teoretycznie potrzebne:
- przechowuje dodatkowy obiekt `PRESENT`
- każdy element to wpis mapy (Entry/Node)

---
5.3 Zachowanie przy kolizjach
- dokładnie takie samo jak w `HashMap`
- od Java 8: lista → drzewo czerwono-czarne przy dużej liczbie kolizji

---
#### 6. Iteracja

Iteracja po `HashSet`:

```java
for (String s : set) {
    System.out.println(s);
} 
```

Pod spodem:

- iteracja po `map.keySet()`

---

#### 7. Usuwanie elementów
```java set.remove("A");```

Pod spodem:

```java map.remove("A");```

---

#### 8. contains()

```java set.contains("A");```

Pod spodem:

```java map.containsKey("A");```

---

#### 9. Dlaczego nie używać HashMap zamiast HashSet?

Teoretycznie można:

```java Map<String, Boolean> map = new HashMap<>();```

Ale:

- semantyka jest niepoprawna (mapa ≠ zbiór)
- większa podatność na błędy
- brak czytelności

---

#### 10. Edge cases i pułapki

10.1 Mutable keys

```java Set<Person> set = new HashSet<>();
Person p = new Person("Jan");

set.add(p);
p.setName("Adam");

set.contains(p); // może zwrócić false
```

Dlaczego:

- zmienia się hashCode → inny bucket

--- 

10.2 Słaby hashCode()

- dużo kolizji

- degradacja wydajności do O(n)

---

10.3 equals() niezgodny z hashCode()
- elementy mogą się "duplikować"
- contains() może nie działać
---

#### 11. Różnice koncepcyjne
| Cecha          | HashSet              | HashMap              |
|----------------|----------------------|----------------------|
| przechowywanie | unikalne elementy   | klucz → wartość     |
| duplikaty      | brak                 | brak dla kluczy     |
| dostęp         | contains             | get                  |
| implementacja  | oparta o HashMap     | natywna              |

#### 12. Pytania rekrutacyjne (mid)
1. Jak HashSet działa wewnętrznie?

Odpowiedź:
HashSet używa HashMap. Elementy są przechowywane jako klucze, a wartość to stały obiekt (PRESENT).

2. Dlaczego HashSet nie pozwala na duplikaty?

Odpowiedź:
Ponieważ HashMap nie pozwala na duplikaty kluczy.

3. Czy HashSet przechowuje wartości?

Odpowiedź:
Nie – przechowuje tylko klucze, a wartości są atrapą.

4. Czy HashSet może zawierać null?

Odpowiedź:
Tak – jeden element null (bo HashMap pozwala na jeden null key).

5. Co się stanie przy złym hashCode?

Odpowiedź:
Wydajność spada (więcej kolizji), może dojść do O(n).

6. Jak działa contains() w HashSet?

Odpowiedź:
Deleguje do HashMap.containsKey().

7. Czy HashSet i HashMap mają taką samą złożoność?

Odpowiedź:
Tak, ponieważ HashSet korzysta z HashMap.

8. Dlaczego HashSet zużywa więcej pamięci niż lista?

Odpowiedź:
Bo używa struktury mapy (buckety + node + wartość).

#### 13. Podsumowanie

Najważniejsze wnioski:

HashSet to wrapper nad HashMap
elementy zbioru = klucze mapy
brak duplikatów wynika z właściwości HashMap
wydajność i problemy są identyczne jak w HashMap
poprawność działania zależy od:
hashCode()
equals()

To jedno z najczęściej zadawanych zagadnień na rozmowach technicznych i absolutny must-have dla mid Java developera.



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