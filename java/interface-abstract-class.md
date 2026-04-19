# Interfejs vs Klasa Abstrakcyjna w Java — Baza Wiedzy dla Mid Java Developera

## Spis treści

1. Wprowadzenie
2. Definicje i model obiektowy
3. Interfejs (interface)
4. Klasa abstrakcyjna (abstract class)
5. Fundamentalne różnice
6. Kiedy używać interfejsu
7. Kiedy używać klasy abstrakcyjnej
8. Porównanie projektowe i architektoniczne
9. Interfejs vs klasa abstrakcyjna w kontekście SOLID
10. Wpływ zmian od Java 8, Java 9 i nowszych
11. Antywzorce i błędne użycia
12. Przykłady produkcyjne
13. Pytania rekrutacyjne z odpowiedziami
14. Podsumowanie

---

# 1. Wprowadzenie

Jedno z najczęściej pojawiających się pytań rekrutacyjnych dla Java Developera brzmi:

**Kiedy użyć interfejsu, a kiedy klasy abstrakcyjnej?**

To pytanie nie dotyczy wyłącznie składni.
Dotyczy:

* modelowania domeny,
* kontraktów API,
* dziedziczenia,
* kompozycji,
* polimorfizmu,
* hermetyzacji,
* projektowania architektury.

Różnica między tymi konstrukcjami jest przede wszystkim semantyczna, a dopiero potem techniczna.

---

# 2. Definicje i model obiektowy

## Interfejs

Interfejs definiuje **kontrakt**.

Mówi:

„Każdy obiekt implementujący mnie musi realizować ten zestaw zachowań.”

Przykład:

```java
public interface PaymentProcessor {

    void processPayment(BigDecimal amount);

}
```

Nie definiuje (przynajmniej klasycznie) jak coś działa.

Definiuje:

* co ma istnieć,
* jakie operacje mają być dostępne,
* jakie jest publiczne API.

---

## Klasa abstrakcyjna

Klasa abstrakcyjna definiuje:

* częściowy kontrakt
* częściową implementację
* wspólny stan
* wspólne zachowanie

```java
public abstract class Animal {

    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void sleep() {
        System.out.println("Sleeping");
    }

    public abstract void makeSound();
}
```

Jest to „niekompletna baza”, z której inne klasy dziedziczą.

---

# 3. Interfejs (interface)

## Cechy

## Wielodziedziczenie typów

Java nie wspiera wielodziedziczenia klas:

```java
class A {}
class B {}

// niedozwolone
class C extends A, B {}
```

Ale wspiera wielokrotną implementację interfejsów:

```java
class User implements Serializable, Comparable<User> {

    @Override
    public int compareTo(User o) {
        return 0;
    }
}
```

To fundamentalna różnica.

---

## Interfejs może zawierać

## abstract methods

```java
void execute();
```

---

## default methods (Java 8)

```java
default void log(){
    System.out.println("log");
}
```

Domyślna implementacja.

---

## static methods

```java
static void validate(){
}
```

---

## private methods (Java 9)

```java
private void helper(){
}
```

Używane wewnętrznie przez default methods.

---

## Pola w interfejsie

Każde pole jest automatycznie:

```java
public static final
```

czyli:

```java
interface Config {

    int TIMEOUT = 5000;

}
```

jest równoważne:

```java
public static final int TIMEOUT = 5000;
```

To stała.
Nie stan obiektu.

---

# 4. Klasa abstrakcyjna

## Może zawierać:

* pola instancyjne
* konstruktory
* metody konkretne
* metody abstrakcyjne
* static methods
* final methods

---

## Przykład

```java
public abstract class Employee {

    protected String id;

    public Employee(String id) {
        this.id = id;
    }

    public void login() {
        System.out.println("Logged in");
    }

    public abstract BigDecimal calculateSalary();
}
```

---

## Stan

To kluczowa różnica.

Interfejs nie przechowuje stanu obiektu.

Klasa abstrakcyjna może.

```java
protected String name;
protected int age;
```

To współdzielony stan dziedziczony.

---

# 5. Fundamentalne różnice

| Cecha              | Interfejs     | Klasa abstrakcyjna |
| ------------------ | ------------- | ------------------ |
| Kontrakt           | Tak           | Tak                |
| Implementacja      | Tak (default) | Tak                |
| Stan               | Nie           | Tak                |
| Konstruktor        | Nie           | Tak                |
| Wielodziedziczenie | Tak           | Nie                |
| Pola instancyjne   | Nie           | Tak                |
| final methods      | Nie           | Tak                |
| protected members  | Nie           | Tak                |
| Dziedziczenie      | implements    | extends            |

---

# 6. Kiedy używać interfejsu

## Gdy modelujesz capability (zdolność)

Nie „czym coś jest”, ale:

„co potrafi”.

```java
interface Flyable {}

interface Swimmable {}
```

Duck może:

```java
class Duck implements Flyable, Swimmable {
}
```

---

## Gdy budujesz kontrakt API

```java
public interface UserRepository {

    Optional<User> findById(Long id);

}
```

Implementacje:

```java
JdbcUserRepository
MongoUserRepository
CachedUserRepository
```

To klasyczny przykład.

---

## Dependency Injection

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository){
        this.repository = repository;
    }

}
```

Programowanie do interfejsu.

Nie do implementacji.

---

# 7. Kiedy używać klasy abstrakcyjnej

## Gdy istnieje relacja IS-A

```java
Dog is an Animal
```

```java
abstract class Animal {}

class Dog extends Animal {}
```

To prawdziwe dziedziczenie.

---

## Wspólna implementacja

```java
abstract class BaseController {

    protected void validateToken() {
    }

}
```

Wiele kontrolerów używa tej samej logiki.

---

## Template Method Pattern

```java
abstract class DataProcessor {

    public final void process(){
        read();
        transform();
        save();
    }

    abstract void read();

    abstract void transform();

    abstract void save();
}
```

To bardzo ważny przypadek.

---

# 8. Porównanie projektowe i architektoniczne

## Interfejs = kontrakt

Bardziej abstrakcyjny.

Luźniejsze powiązanie.

Lower coupling.

Większa testowalność.

---

## Klasa abstrakcyjna = częściowa implementacja

Silniejsze powiązanie.

Tighter coupling.

Większa zależność od hierarchii.

---

## Prefer composition over inheritance

Zasada:

Preferuj:

```java
class OrderService {

   private PaymentProcessor processor;

}
```

zamiast:

```java
class OrderService extends PaymentProcessor {
}
```

---

# 9. SOLID

## S — Single Responsibility

Źle:

```java
abstract class SuperBaseManager {
}
```

God class.

Naruszenie SRP.

---

## O — Open Closed

Interfejsy świetnie wspierają OCP.

Dodajesz nową implementację.

Nie zmieniasz kodu.

```java
PaymentProcessor

StripePaymentProcessor
PaypalPaymentProcessor
```

---

## L — Liskov

Klasy abstrakcyjne bardzo łatwo łamią LSP.

Klasyczny problem:

```java
class Bird {

   void fly() {}

}

class Penguin extends Bird {

}
```

Penguin nie lata.

Zła hierarchia.

---

## D — Dependency Inversion

Preferowane:

```java
interface NotificationSender {}
```

nie:

```java
abstract class NotificationSender {}
```

jeśli nie ma potrzeby wspólnego stanu.

---

# 10. Java 8+ zmieniła wszystko

Przed Java 8:

Interfejs:

* tylko abstrakcyjne metody

Klasa abstrakcyjna:

* implementacja

Granica była wyraźna.

---

Po Java 8:

```java
interface Audit {

    default void log(){
    }

}
```

Interfejs zaczął zawierać zachowanie.

To zmniejszyło różnicę.

Ale nie usunęło.

Bo nadal:

* brak stanu
* brak konstruktorów
* brak protected
* brak normal inheritance

---

# 11. Diamond Problem

```java
interface A {

   default void test(){
      System.out.println("A");
   }

}

interface B {

   default void test(){
      System.out.println("B");
   }

}

class X implements A,B {

   @Override
   public void test(){
      A.super.test();
   }

}
```

Java wymusza rozwiązanie konfliktu.

---

# 12. Antywzorce

## Marker interfaces zamiast adnotacji

Stare:

```java
Serializable
Cloneable
```

Dziś często lepiej:

```java
@Audited
```

---

## BaseAbstractEverything

```java
AbstractService
AbstractController
AbstractManager
AbstractProcessor
```

Często symptom złego designu.

---

## Interfejs z jedną implementacją bez powodu

```java
UserService
UserServiceImpl
```

Jeśli nigdy nie będzie drugiej implementacji i nie ma DI/testowalności jako celu — często overengineering.

---

# 13. Przykłady produkcyjne

## Spring

Interfejs:

```java
JpaRepository
CrudRepository
ListCrudRepository
```

---

## Abstract classes

```java
AbstractController
AbstractAuthenticationToken
AbstractRoutingDataSource
```

---

## JDBC

Interfejs:

```java
Connection
Statement
PreparedStatement
```

Implementacje dostarcza driver.

---

# 14. Interfejs czy abstract — heurystyka

Zadaj 5 pytań.

## Czy potrzebuję wspólnego stanu?

Tak:

Abstract.

---

## Czy potrzebuję wielu implementacji?

Tak:

Interface.

---

## Czy modeluję capability?

Tak:

Interface.

---

## Czy potrzebuję współdzielonego kodu?

Tak:

Abstract.

---

## Czy chcę luźne powiązanie?

Tak:

Interface.

---

# 15. Pytania rekrutacyjne z odpowiedziami

## Pytanie 1

Czym różni się interfejs od klasy abstrakcyjnej?

## Odpowiedź

Interfejs definiuje kontrakt.

Klasa abstrakcyjna definiuje kontrakt plus częściową implementację oraz wspólny stan.

Interfejs wspiera multiple inheritance of type.

Klasa abstrakcyjna nie.

---

## Pytanie 2

Czy interfejs może mieć implementację?

## Odpowiedź

Tak.

Od Java 8:

* default methods
* static methods

Od Java 9:

* private methods

---

## Pytanie 3

Czy interfejs może mieć konstruktor?

## Odpowiedź

Nie.

Interfejs nie ma stanu obiektu.

Nie może być instancjonowany.

Nie potrzebuje konstruktora.

---

## Pytanie 4

Czy klasa abstrakcyjna może nie mieć metod abstrakcyjnych?

## Odpowiedź

Tak.

```java
abstract class A {

   void test(){
   }

}
```

Nadal poprawne.

abstract może blokować tworzenie instancji.

---

## Pytanie 5

Czy można implementować wiele interfejsów i rozszerzać jedną klasę abstrakcyjną?

## Odpowiedź

Tak.

```java
class X extends Base implements A,B,C {
}
```

To bardzo częsty pattern.

---

## Pytanie 6

Dlaczego w Spring preferuje się interfejsy?

## Odpowiedź

Bo wspierają:

* Dependency Injection
* Proxy
* Mockowanie
* AOP
* Loose coupling
* OCP

---

## Pytanie 7

Czy default methods łamią sens interfejsów?

## Odpowiedź

Nie.

Rozszerzają kontrakt o behavior.

Głównie dla backward compatibility.

Np. ewolucja API.

---

## Pytanie 8

Czy interfejs może zastąpić abstract class?

## Odpowiedź

Nie zawsze.

Jeśli potrzebujesz:

* state
* constructors
* protected fields
* template method
* shared code

wtedy potrzebujesz abstract class.

---

## Pytanie 9

Co jest lepsze — interfejs czy abstract?

## Odpowiedź

To błędne pytanie.

To narzędzia do innych problemów.

Lepsze pytanie:

Jaki problem modeluję?

---

## Pytanie 10

Co preferujesz na mid/senior poziomie?

## Odpowiedź

Domyślnie:

* prefer interface
* use abstract when shared state/behavior required
* prefer composition over inheritance

To najczęściej uznawana dobra praktyka.

---

# 16. Tricky pytania rekrutacyjne

## Czy interfejs może dziedziczyć po interfejsie?

Tak.

```java
interface A {}

interface B extends A {}
```

---

## Czy abstract class może implementować interfejs?

Tak.

```java
abstract class BaseService implements Service {
}
```

---

## Czy abstract class może nie implementować wszystkich metod interfejsu?

Tak.

Jeśli sama jest abstract.

---

## Czy można mieć final abstract class?

Nie.

Sprzeczność.

* final -> nie można dziedziczyć
* abstract -> trzeba dziedziczyć

Compiler error.

---

# 17. Najczęstsza odpowiedź rekrutacyjna (wersja idealna)

Jeżeli modeluję kontrakt lub capability i zależy mi na luźnym powiązaniu, wybieram interfejs.

Jeżeli mam wspólny stan, współdzieloną implementację albo Template Method, wybieram klasę abstrakcyjną.

Domyślnie preferuję interfejsy i kompozycję ponad dziedziczenie.

To bardzo mocna odpowiedź na rozmowie.

---

# 18. Podsumowanie

## Użyj interfejsu gdy:

* definiujesz kontrakt
* chcesz DI
* chcesz mockowanie
* chcesz wiele implementacji
* modelujesz capability
* chcesz loose coupling

---

## Użyj klasy abstrakcyjnej gdy:

* potrzebujesz stanu
* potrzebujesz konstruktora
* masz wspólny kod
* używasz Template Method
* masz realne IS-A

---

## Zasada praktyczna

```text
Prefer Interface
Use Abstract Class only when needed
Prefer Composition over Inheritance
```

To jest podejście oczekiwane od mid/senior Java Developera.

---

# 19. Dodatkowe tematy powiązane do nauki

Następne naturalne rozszerzenia tego tematu:

* Composition vs Inheritance
* Liskov Substitution Principle
* Default Methods i Diamond Problem
* Strategy Pattern
* Template Method Pattern
* Dependency Inversion Principle
* Marker Interfaces vs Annotations
* Sealed Interfaces (Java 17+)
* Functional Interfaces
* Records + Interfaces
