# Pamięć w aplikacjach: stack, heap oraz garbage collector

## Wprowadzenie

Zrozumienie działania pamięci jest obowiązkowe na poziomie mid developera. Problemy wydajnościowe, memory leak, nieprzewidywalne pauzy aplikacji czy stack overflow bardzo często wynikają z braku świadomości, jak działa:

* stack
* heap
* garbage collector

W zależności od języka programowania (na przykład Java, C++, C#, Python) zarządzanie pamięcią może być:

* manualne (C plus plus),
* automatyczne z użyciem garbage collector (Java, C Sharp, Python),
* oparte o model ownership (na przykład Rust).

---

# 1. Stack

## Czym jest stack?

Stack to obszar pamięci zarządzany w sposób automatyczny, działający w modelu LIFO (Last In First Out).

Na stack trafiają:

* zmienne lokalne,
* parametry funkcji,
* adres powrotu z funkcji,
* informacje o kontekście wywołania.

Każde wywołanie funkcji powoduje utworzenie stack frame. Po zakończeniu funkcji frame jest zdejmowany.

---

## Jak działa stack?

Rozważ przykład w języku Java:

```java
public void example() {
    int a = 5;
    int b = 10;
    int result = add(a, b);
}

public int add(int x, int y) {
    int sum = x + y;
    return sum;
}
```

Na stack trafiają:

* `a`, `b`, `result`,
* `x`, `y`,
* `sum`,
* adres powrotu.

Ważne: w przypadku obiektów na stack przechowywana jest referencja, a nie sam obiekt.

---

## Cechy stack

* bardzo szybki dostęp,
* brak fragmentacji pamięci,
* ograniczony rozmiar,
* automatyczne czyszczenie po wyjściu z zakresu.

---

## Dobre praktyki

* unikać głębokiej rekurencji bez kontroli warunku stopu,
* nie alokować dużych struktur jako zmiennych lokalnych,
* świadomie projektować wywołania rekurencyjne,
* analizować potencjalne ryzyko stack overflow przy pracy z drzewami lub grafami.

---

## Najczęstsze błędy

* brak warunku stopu w rekurencji,
* przetwarzanie bardzo głębokich struktur rekurencyjnie,
* ignorowanie limitu stack w środowisku produkcyjnym.

---

# 2. Heap

## Czym jest heap?

Heap to obszar pamięci przeznaczony do dynamicznej alokacji obiektów w czasie działania aplikacji.

Obiekty tworzone przy użyciu `new` trafiają na heap.

Przykład w Java:

```java
User user = new User("Jan", "Kowalski");
```

* referencja `user` znajduje się na stack,
* instancja `User` znajduje się na heap.

---

## Cechy heap

* większy rozmiar niż stack,
* wolniejszy dostęp niż stack,
* podatność na fragmentację,
* wymaga zarządzania pamięcią (manualnie lub przez garbage collector).

---

## Cykl życia obiektu na heap

1. Alokacja pamięci.
2. Użycie przez aplikację.
3. Brak referencji.
4. Usunięcie przez garbage collector (w językach zarządzanych).

---

## Dobre praktyki

* ograniczać czas życia obiektów,
* usuwać referencje, które nie są już potrzebne,
* unikać długowiecznych statycznych kolekcji,
* stosować pooling obiektów w miejscach o bardzo wysokiej alokacji,
* analizować heap dump przy problemach produkcyjnych.

---

## Najczęstsze błędy

* memory leak w aplikacjach z garbage collector,
* trzymanie referencji w cache bez limitu,
* tworzenie obiektów w każdej iteracji pętli bez potrzeby,
* brak zrozumienia różnicy między referencją a obiektem.

---

# 3. Garbage collector

## Czym jest garbage collector?

Garbage collector to mechanizm automatycznego usuwania obiektów z heap, do których nie istnieją już żadne osiągalne referencje.

Występuje między innymi w:

* Java
* C#
* Python

---

## Jak działa garbage collector?

Najczęściej stosowany jest model generational.

Heap jest podzielony na:

* Young Generation – nowe obiekty,
* Old Generation – obiekty długowieczne,
* Metaspace (w nowszych wersjach Java).

Założenie: większość obiektów żyje krótko.

---

## Główne algorytmy

* Mark and Sweep
* Mark and Compact
* Generational Collection

Uproszczony proces:

1. Mark – oznaczanie obiektów osiągalnych z tzw. root set.
2. Sweep – usuwanie nieosiągalnych.
3. Compact – opcjonalne przesunięcie obiektów w celu redukcji fragmentacji.

---

## Stop The World

W wielu implementacjach garbage collector zatrzymuje wykonywanie aplikacji na czas czyszczenia pamięci. Długie pauzy mogą powodować problemy wydajnościowe w systemach czasu rzeczywistego.

---

## Dobre praktyki

* projektować obiekty o krótkim cyklu życia,
* unikać silnych referencji w globalnych strukturach,
* stosować weak reference, gdy uzasadnione,
* monitorować metryki pamięci w środowisku produkcyjnym,
* testować aplikację pod obciążeniem.

---

## Najczęstsze błędy

* przekonanie, że garbage collector eliminuje memory leak,
* brak profilowania pamięci,
* ignorowanie wysokiej częstotliwości uruchamiania garbage collector,
* używanie finalizerów jako mechanizmu zwalniania zasobów.

---

# 4. Stack vs Heap

| Cecha        | Stack                       | Heap                           |
| ------------ | --------------------------- | ------------------------------ |
| Zarządzanie  | Automatyczne                | Manualne lub garbage collector |
| Szybkość     | Bardzo szybki               | Wolniejszy                     |
| Rozmiar      | Ograniczony                 | Znacznie większy               |
| Czas życia   | Związany z zakresem funkcji | Zależny od referencji          |
| Fragmentacja | Brak                        | Możliwa                        |

---

# 5. Pytania rekrutacyjne z odpowiedziami

## 1. Dlaczego aplikacja może mieć memory leak mimo użycia garbage collector?

**Odpowiedź:**
Jeżeli obiekt nadal jest osiągalny z root set (na przykład przez statyczną kolekcję), garbage collector go nie usunie, nawet jeśli logicznie nie jest już potrzebny.

---

## 2. Co powoduje stack overflow?

**Odpowiedź:**
Zbyt głęboka rekurencja lub bardzo głęboki łańcuch wywołań funkcji powodują przekroczenie limitu pamięci stack.

---

## 3. Czym różni się referencja od obiektu?

**Odpowiedź:**
Referencja to wskaźnik przechowywany zwykle na stack, który wskazuje na obiekt znajdujący się na heap. Sam obiekt zawiera dane.

---

## 4. Dlaczego częsta alokacja w pętli jest kosztowna?

**Odpowiedź:**
Generuje dużą presję na heap i zwiększa częstotliwość działania garbage collector, co może prowadzić do pauz aplikacji.

---

## 5. Dlaczego model generational poprawia wydajność?

**Odpowiedź:**
Ponieważ większość obiektów żyje krótko, czyszczenie Young Generation jest szybkie i pozwala uniknąć częstego skanowania całego heap.

---

# Podsumowanie

Świadome rozumienie:

* stack,
* heap,
* garbage collector,

pozwala:

* diagnozować problemy wydajnościowe,
* unikać memory leak,
* projektować przewidywalny cykl życia obiektów,
* optymalizować zużycie pamięci w aplikacjach produkcyjnych.

To fundament wiedzy wymagany na poziomie mid developera i bardzo częsty obszar weryfikowany podczas rozmów technicznych.
