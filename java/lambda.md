# Baza wiedzy Java (Mid Developer) — Lambdy

## Spis treści

1. Wprowadzenie i motywacja
2. Czym jest lambda w Javie
3. Składnia i semantyka
4. Functional Interfaces (interfejsy funkcyjne)
5. Typowanie, target typing i inferencja
6. Przechwytywanie zmiennych (closures)
7. Lambdy a anonymous classes
8. Method References
9. Lambdy i Stream API
10. Domyślne metody (default methods) a lambdy
11. Checked exceptions i obsługa wyjątków
12. Wydajność, bytecode i invokedynamic
13. Pułapki i dobre praktyki
14. Wzorce użycia w kodzie produkcyjnym
15. Najczęstsze błędy
16. Pytania rekrutacyjne z odpowiedziami
17. Zadania praktyczne

---

# 1. Wprowadzenie i motywacja

Lambdy zostały wprowadzone w Java 8 jako element programowania funkcyjnego.

Ich główny cel:

* redukcja boilerplate code,
* umożliwienie przekazywania zachowania (behavior) jako argumentu,
* wsparcie dla Stream API,
* poprawa czytelności operacji kolekcyjnych,
* implementacja stylu deklaratywnego.

Przed Java 8 dominowało podejście obiektowe z:

* anonymous inner classes,
* Strategy Pattern,
* Command Pattern,
* verbose callback APIs.

Przykład przed lambdami:

```java
Collections.sort(users, new Comparator<User>() {
    @Override
    public int compare(User a, User b) {
        return a.getName().compareTo(b.getName());
    }
});
```

Po Java 8:

```java
Collections.sort(users,
    (a,b) -> a.getName().compareTo(b.getName())
);
```

---

# 2. Czym jest lambda w Javie

Lambda expression to anonimowa funkcja:

* bez nazwy,
* może być przekazana jako argument,
* może być zwrócona z metody,
* może zostać przypisana do zmiennej typu functional interface.

Ogólna postać:

```java
(parameters) -> expression
```

lub

```java
(parameters) -> {
   statements
}
```

Przykład:

```java
(x, y) -> x + y
```

To implementacja zachowania dla interfejsu:

```java
@FunctionalInterface
interface Calculator {
   int add(int a, int b);
}
```

Lambda:

```java
Calculator c = (a,b) -> a+b;
```

---

# 3. Składnia i semantyka

## Bez parametrów

```java
() -> System.out.println("Hello")
```

## Jeden parametr

```java
x -> x * 2
```

lub:

```java
(int x) -> x * 2
```

## Wiele parametrów

```java
(a,b) -> a+b
```

## Wielolinijkowa lambda

```java
(a,b) -> {
   int result = a+b;
   return result;
}
```

## Return

Dla pojedynczego expression:

```java
x -> x+1
```

return jest implicit.

Dla block:

```java
x -> {
   return x+1;
}
```

return wymagany.

---

# 4. Functional Interfaces

## Definicja

Interfejs z dokładnie jedną metodą abstrakcyjną.

Może mieć:

* default methods
* static methods
* metody Object

Nie liczą się do limitu jednej abstrakcyjnej.

## @FunctionalInterface

```java
@FunctionalInterface
interface Printer {
   void print(String s);
}
```

Kompilator wymusi poprawność.

---

## Standardowe functional interfaces

## Predicate<T>

```java
Predicate<String> p = s -> s.isEmpty();
```

Metoda:

```java
boolean test(T t)
```

---

## Function<T,R>

```java
Function<String,Integer> f=
    s->s.length();
```

```java
R apply(T t)
```

---

## Consumer<T>

```java
Consumer<String> c=
 s->System.out.println(s);
```

```java
void accept(T t)
```

---

## Supplier<T>

```java
Supplier<User> s=()->new User();
```

```java
T get()
```

---

## BiFunction

```java
BiFunction<Integer,Integer,Integer> sum=
 (a,b)->a+b;
```

---

## UnaryOperator

```java
UnaryOperator<String> upper=
 s->s.toUpperCase();
```

---

## BinaryOperator

```java
BinaryOperator<Integer> max=
 Integer::max;
```

---

# 5. Target Typing i inferencja

Lambda sama nie ma typu.

Ma target type.

```java
Predicate<String> p=s->s.isEmpty();
```

Typ wynika z lewej strony.

## Overload ambiguity

Problem:

```java
void doIt(Consumer<String> c)
void doIt(Predicate<String> p)
```

```java
doIt(s -> s.length());
```

Ambiguous.

Wymaga cast:

```java
doIt((Predicate<String>) s->s.isEmpty());
```

---

# 6. Closures i effectively final

Lambda może przechwytywać zmienne lokalne.

```java
int x=10;
Function<Integer,Integer> f=n->n+x;
```

x jest captured.

## effectively final

To:

```java
int x=10;
```

jest OK.

To:

```java
int x=10;
x++;
```

już nie.

Powód:

Lambda przechwytuje kopię, nie referencję do lokalnej zmiennej ze stack.

To unika:

* race conditions
* visibility issues
* lifetime problems

---

# 7. Lambda vs Anonymous Inner Class

## Anonymous class

```java
Runnable r=new Runnable(){
 public void run(){
   System.out.println(this);
 }
};
```

## Lambda

```java
Runnable r=()->{
 System.out.println(this);
};
```

Różnica:

W anonymous class:

```java
this
```

odnosi się do obiektu anonymous class.

W lambda:

```java
this
```

odnosi się do otaczającego obiektu.

To fundamentalna różnica.

---

# 8. Method References

Skrócona forma lambd.

## Static method

```java
Integer::parseInt
```

zamiast:

```java
s->Integer.parseInt(s)
```

---

## Instance method konkretnego obiektu

```java
printer::print
```

---

## Instance method arbitralnego obiektu

```java
String::toUpperCase
```

---

## Constructor reference

```java
User::new
```

---

# 9. Lambdy i Stream API

## filter

```java
users.stream()
 .filter(u->u.isActive())
```

---

## map

```java
.map(User::getEmail)
```

---

## reduce

```java
.reduce(0,(a,b)->a+b)
```

---

## collect

```java
.collect(Collectors.toList())
```

---

Pipeline:

```java
List<String> result=
users.stream()
 .filter(u->u.isActive())
 .map(User::getName)
 .sorted()
 .toList();
```

---

# 10. Default methods

```java
interface A {

 default void log(){
 }

 void run();
}
```

Nie łamie functional interface.

---

# 11. Checked Exceptions

Problem:

```java
list.forEach(s-> {
 throw new IOException();
});
```

Błąd.

Consumer nie deklaruje throws.

## Wrapper

```java
static Consumer<String>
wrap(ThrowingConsumer<String> tc){
 return s->{
   try{
      tc.accept(s);
   }catch(Exception e){
      throw new RuntimeException(e);
   }
 };
}
```

Częsty pattern.

---

# 12. Wydajność, Bytecode, invokedynamic

To bardzo ważne rekrutacyjnie.

Lambda NIE jest kompilowana do anonymous class.

To częsty mit.

Kompilator generuje:

```java
invokedynamic
```

z wykorzystaniem:

```java
LambdaMetafactory
```

JVM może:

* optymalizować
* inline’ować
* cache'ować instancje non-capturing lambdas

## Non-capturing lambda

```java
x -> x+1
```

może być singleton.

---

## Capturing lambda

```java
x -> x+external
```

nowa instancja może powstawać per invocation.

---

# 13. Pułapki

## Side effects

Źle:

```java
List<String> names=new ArrayList<>();

users.stream()
 .forEach(u->names.add(u.getName()));
```

Mutacja.

Lepsze:

```java
List<String> names=
users.stream()
 .map(User::getName)
 .toList();
```

---

## Parallel stream i mutable state

Źle:

```java
List<Integer> list=new ArrayList<>();

IntStream.range(0,1000)
.parallel()
.forEach(list::add);
```

Race condition.

---

## Overusing lambdas

Nie każda logika powinna być lambdą.

Źle:

```java
x->{
 if(...)
   ...
 else if(...)
   ...
 else
   ...
}
```

Za dużo logiki.

---

# 14. Wzorce użycia

## Strategy

```java
interface Discount {
 BigDecimal apply(BigDecimal p);
}
```

```java
Discount d= p->p.multiply(
 new BigDecimal("0.9")
);
```

---

## Retry

```java
<T> T retry(Supplier<T> op)
```

---

## Callback

```java
execute(()->service.call())
```

---

# 15. Najczęstsze błędy

## Błąd 1

Mylenie lambda z closure z JavaScript.

Java ma ograniczone closures.

---

## Błąd 2

Myślenie że lambda = anonymous class.

Fałsz.

---

## Błąd 3

Mutacje w streamach.

---

## Błąd 4

Ignorowanie effectively final.

---

# 16. Pytania rekrutacyjne z odpowiedziami

## Pytanie 1

Co to jest functional interface?

## Odpowiedź

Interfejs posiadający dokładnie jedną metodę abstrakcyjną.
Może zawierać default i static methods.
Jest target type dla lambda expressions.

---

## Pytanie 2

Czy lambda jest anonymous inner class?

## Odpowiedź

Nie.

Lambda kompiluje się z użyciem:

* invokedynamic
* LambdaMetafactory

Nie jako odrębna anonymous class.

---

## Pytanie 3

Czym jest effectively final?

## Odpowiedź

Zmienna nie musi mieć final, ale nie może zostać zmodyfikowana po inicjalizacji.
To wymagane dla captured variables.

---

## Pytanie 4

Dlaczego Java wymaga effectively final?

## Odpowiedź

Bo lokalne zmienne są na stack.
Lambda może żyć dłużej niż frame metody.
Przechwytywana jest wartość.
Zapobiega to problemom concurrency i visibility.

---

## Pytanie 5

Różnica między:

```java
this
```

w lambda i anonymous class?

## Odpowiedź

W lambda:

odnosi się do otaczającej klasy.

W anonymous class:

odnosi się do instancji anonymous class.

---

## Pytanie 6

Co to target typing?

## Odpowiedź

Typ lambdy wynika z kontekstu.
Np:

```java
Predicate<String> p=s->true;
```

Predicate determinuje sygnaturę.

---

## Pytanie 7

Co jest problemem z checked exceptions w lambdach?

## Odpowiedź

Standardowe functional interfaces nie deklarują throws.
Dlatego checked exceptions wymagają wrapperów lub custom interfaces.

---

## Pytanie 8

Difference between Function and Consumer?

## Odpowiedź

Function:

* przyjmuje argument
* zwraca wynik

Consumer:

* przyjmuje argument
* nie zwraca wyniku.

---

## Pytanie 9

Czy lambdy są zawsze szybsze?

## Odpowiedź

Nie.

Często JVM optymalizuje je bardzo dobrze.
Ale capturing lambdas, boxing/unboxing albo złe użycie streamów może pogorszyć wydajność.

---

## Pytanie 10

Co robi:

```java
String::toUpperCase
```

## Odpowiedź

To method reference do instance method arbitralnego obiektu typu String.

---

## Pytanie 11

Czym różni się:

```java
map()
```

od:

```java
flatMap()
```

## Odpowiedź

map:

transformuje 1->1.

flatMap:

transformuje 1->many i spłaszcza strukturę.

---

## Pytanie 12

Co dzieje się pod spodem z lambda na poziomie JVM?

## Odpowiedź

Bytecode używa:

```java
invokedynamic
```

Bootstrap method:

```java
LambdaMetafactory
```

Runtime tworzy implementację functional interface.

To zaawansowane pytanie bardzo często pada na mid/senior.

---

# 17. Zadania praktyczne

## Zadanie 1

Napisz własny functional interface:

```java
@FunctionalInterface
interface Validator<T>{
 boolean validate(T t);
}
```

---

## Zadanie 2

Zamień:

```java
Collections.sort(list,
new Comparator<Integer>(){
 @Override
 public int compare(Integer a,Integer b){
   return a-b;
 }
});
```

na lambda.

Rozwiązanie:

```java
Collections.sort(list,
(a,b)->a-b);
```

---

## Zadanie 3

Zrób wrapper dla checked exceptions.

To bardzo częsty live coding.

---

# Checklist Mid Java Developer

Powinieneś umieć wyjaśnić:

* functional interfaces
* Predicate/Function/Consumer/Supplier
* closures
* effectively final
* target typing
* method references
* invokedynamic
* LambdaMetafactory
* lambda vs anonymous class
* lambdy w Stream API
* checked exceptions
* pułapki concurrency

Jeśli nie umiesz odpowiedzieć o:

* invokedynamic
* this w lambda
* effectively final

bardzo często jesteś oceniany poniżej poziomu solid Mid.

---

# Podsumowanie

Lambda w Javie:

* nie jest tylko skrótem składniowym,
* jest oparta o functional interfaces,
* jest implementowana przez invokedynamic,
* wspiera styl funkcyjny,
* ma istotne konsekwencje dla wydajności, semantyki i concurrency.

To jedno z fundamentalnych zagadnień Java Mid/Senior interview.
