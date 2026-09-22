# Functional interface ve method reference

Lambda ifadesi yazmak için önce hedef tipin tam olarak ne olduğu bilinmelidir: Java'da lambda, tek soyut metodu (`SAM`) olan bir arayüze -- yani `functional interface`'e -- atanabilir. Bazen bu lambda gövdesi zaten var olan bir metodu çağırmaktan başka bir şey yapmaz; bu durumda method reference (`::`) sözdizimi hem daha kısa hem daha okunur bir alternatif sunar.

## Functional interface tanımı ve lambda ataması

```java
@FunctionalInterface
interface Transformer<T, R> {
    R apply(T input);
}

Transformer<String, Integer> length = s -> s.length();
Transformer<String, String> upper = String::toUpperCase;

System.out.println(length.apply("merhaba"));
System.out.println(upper.apply("merhaba"));
```

`@FunctionalInterface` anotasyonu zorunlu değildir ama derleyiciye arayüzün tam olarak bir soyut metot içermesi gerektiğini bildirir; ikinci bir soyut metot eklenirse derleme hatası alınır. `default` ve `static` metotlar bu kurala dahil değildir, sadece soyut metot sayısı önemlidir. `java.util.function` paketindeki `Function`, `Predicate`, `Supplier`, `Consumer` gibi hazır arayüzler çoğu senaryoda özel arayüz tanımlamaya gerek bırakmaz.

## Dört method reference türü

```java
Supplier<ArrayList<String>> ctor = ArrayList::new;          // constructor reference
Function<String, Integer> parse = Integer::parseInt;         // static method reference
Function<String, String> trim = String::trim;                 // unbound instance method
String prefix = "log: ";
Function<String, String> withPrefix = prefix::concat;         // bound instance method
```

`Integer::parseInt` bir `static` metoda referans verirken, `String::trim` sınıfa değil örneğe bağlı bir metodu ifade eder; buradaki fark, `trim`'in çağrılacağı örneğin parametre olarak lambda'ya sonradan geleceğidir (`unbound`). `prefix::concat` ise `prefix` değişkenine çoktan bağlanmış (`bound`) bir örnek metodudur, sadece ikinci argüman dışarıdan gelir. `ArrayList::new` gibi constructor reference'lar `Supplier` veya benzer arayüzlerle nesne üretimini lambda yazmadan ifade eder.

## Ne zaman işe yarar

- Lambda gövdesi sadece var olan bir metodu çağırıyorsa (`s -> s.toUpperCase()` yerine `String::toUpperCase`), method reference niyeti daha net gösterir.
- `Comparator.comparing(Person::getAge)` gibi Stream/Comparator API'leriyle birleştiğinde kod önemli ölçüde kısalır.
- Aşırı yüklenmiş (`overloaded`) metotlarda method reference hedef tipi belirsizleştirebilir; böyle durumlarda açık lambda daha güvenlidir.

## Denemek için

```
jshell> interface Greeter { String greet(String name); }
jshell> Greeter g = "Merhaba, %s!"::formatted
jshell> g.greet("Dünya")
```

`"Merhaba, %s!"::formatted` ifadesinin bound instance method reference olduğunu ve `String.formatted(Object...)` metoduna karşılık geldiğini gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/FunctionalInterface.html]
