# Java Stream API – Baza Wiedzy (Mid Developer)

## Spis treści

1. Wprowadzenie
2. Podstawowe pojęcia
3. Pipeline przetwarzania
4. Operacje pośrednie (Intermediate)
5. Operacje końcowe (Terminal)
6. Lazy evaluation i short-circuiting
7. Równoległość (Parallel Streams)
8. Kolektory (Collectors)
9. Pułapki i dobre praktyki
10. Porównanie z podejściem imperatywnym
11. Przykłady zaawansowane
12. Pytania rekrutacyjne (z odpowiedziami)

---

## 1. Wprowadzenie

Stream API zostało wprowadzone w Java 8 jako element programowania funkcyjnego. Umożliwia deklaratywne przetwarzanie kolekcji danych poprzez operacje takie jak filtrowanie, mapowanie czy redukcja.

Stream NIE jest strukturą danych – jest abstrakcją przepływu danych.

---

## 2. Podstawowe pojęcia

### Stream

Strumień danych, który reprezentuje sekwencję elementów wspierającą operacje przetwarzania.

### Źródło danych

* Kolekcje (List, Set)
* Tablice
* I/O (np. Files.lines)
* Generatory (Stream.generate, Stream.iterate)

### Niezmienność

Operacje na streamach NIE modyfikują źródła danych.

---

## 3. Pipeline przetwarzania

Pipeline składa się z:

1. Źródła
2. Operacji pośrednich
3. Operacji końcowej

Przykład:

```java
List<String> result = list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 4. Operacje pośrednie (Intermediate)

Charakterystyka:

* Zwracają nowy Stream
* Są leniwe (lazy)

### Najważniejsze operacje

#### filter

Filtruje elementy:

```java
stream.filter(x -> x > 10);
```

#### map

Transformuje elementy:

```java
stream.map(x -> x * 2);
```

#### flatMap

Spłaszcza strukturę:

```java
list.stream()
    .flatMap(Collection::stream);
```

#### distinct

Usuwa duplikaty (opiera się na equals/hashCode)

#### sorted

Sortowanie:

```java
stream.sorted(Comparator.naturalOrder());
```

#### limit / skip

Operacje ograniczające:

```java
stream.limit(10).skip(5);
```

---

## 5. Operacje końcowe (Terminal)

Charakterystyka:

* Zwracają wynik
* Uruchamiają pipeline

### Przykłady

#### forEach

```java
stream.forEach(System.out::println);
```

#### collect

```java
stream.collect(Collectors.toList());
```

#### reduce

```java
int sum = stream.reduce(0, Integer::sum);
```

#### findFirst / findAny

Zwracają Optional

#### anyMatch / allMatch / noneMatch

Operacje predykatowe

---

## 6. Lazy evaluation i short-circuiting

Operacje pośrednie są wykonywane dopiero w momencie wywołania operacji końcowej.

### Przykład

```java
list.stream()
    .filter(x -> {
        System.out.println("filter: " + x);
        return x > 2;
    })
    .map(x -> {
        System.out.println("map: " + x);
        return x * 2;
    })
    .findFirst();
```

Pipeline zakończy się wcześniej dzięki short-circuiting.

---

## 7. Równoległość (Parallel Streams)

```java
list.parallelStream()
    .map(x -> x * 2)
    .forEach(System.out::println);
```

### Cechy

* Wykorzystuje ForkJoinPool
* Brak gwarancji kolejności
* Operacje muszą być stateless i side-effect free

### Problemy

* Overhead
* Race conditions
* Synchronizacja

---

## 8. Kolektory (Collectors)

### Najczęstsze

#### toList

```java
Collectors.toList()
```

#### groupingBy

```java
Collectors.groupingBy(Person::getAge)
```

#### partitioningBy

```java
Collectors.partitioningBy(x -> x > 10)
```

#### joining

```java
Collectors.joining(", ")
```

#### counting

```java
Collectors.counting()
```

---

## 9. Pułapki i dobre praktyki

### Pułapki

* Używanie streamów do prostych operacji
* Nadużywanie parallelStream
* Mutowalne obiekty
* Side effects

### Dobre praktyki

* Preferuj immutability
* Używaj method references
* Dbaj o czytelność

---

## 10. Porównanie z podejściem imperatywnym

### Imperatywnie

```java
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 3) {
        result.add(s.toUpperCase());
    }
}
```

### Stream API

```java
list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 11. Przykłady zaawansowane

### Grupowanie i mapowanie

```java
Map<Integer, List<String>> result = people.stream()
    .collect(Collectors.groupingBy(
        Person::getAge,
        Collectors.mapping(Person::getName, Collectors.toList())
    ));
```

### Redukcja z obiektem

```java
Optional<Person> oldest = people.stream()
    .reduce((p1, p2) -> p1.getAge() > p2.getAge() ? p1 : p2);
```

---

## 12. Pytania rekrutacyjne (z odpowiedziami)

### 1. Czym różni się map od flatMap?

**Odpowiedź:**
map przekształca element 1:1, natomiast flatMap służy do spłaszczania struktur (np. Stream<List<T>> -> Stream<T>).

---

### 2. Co to jest lazy evaluation?

**Odpowiedź:**
Operacje są wykonywane dopiero przy operacji końcowej, co pozwala na optymalizację i short-circuiting.

---

### 3. Dlaczego parallelStream może być niebezpieczny?

**Odpowiedź:**
Może prowadzić do problemów z współbieżnością, race conditions oraz większego narzutu niż zysków.

---

### 4. Czym różni się findFirst od findAny?

**Odpowiedź:**
findFirst zachowuje kolejność, findAny może zwrócić dowolny element (bardziej wydajny w parallel stream).

---

### 5. Co to jest collector?

**Odpowiedź:**
Obiekt definiujący sposób agregacji elementów streama do struktury wynikowej.

---

### 6. Czy stream można użyć ponownie?

**Odpowiedź:**
Nie. Stream jest jednorazowy – po operacji terminalnej zostaje zamknięty.

---

### 7. Co to jest reduce?

**Odpowiedź:**
Operacja agregująca elementy do jednej wartości przy użyciu funkcji akumulującej.

---

## Podsumowanie

Stream API to potężne narzędzie do deklaratywnego przetwarzania danych. Kluczowe jest zrozumienie:

* leniwości (lazy evaluation)
* pipeline
* różnicy między operacjami
* konsekwencji użycia parallelStream

Dla poziomu mid wymagane jest świadome użycie, a nie tylko znajomość składni.
