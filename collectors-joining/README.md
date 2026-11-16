# Collectors.joining ile stream'den tek bir String üretme

Bir `Stream<String>`'i tek bir metinde birleştirmek istendiğinde elle yazılan döngü, ayırıcıyı her eleman sonrasına eklemekten kaçınmak için fazladan bir koşul ya da `StringBuilder.deleteCharAt` gibi bir düzeltme gerektirir. `Collectors.joining`, bu ayırıcı/önek/sonek mantığını stream'in `collect` adımına taşıyarak elle yönetilen durumu ortadan kaldırır.

## Temel kullanım: ayırıcı ile birleştirme

```java
List<String> fruits = List.of("elma", "armut", "kiraz");

String joined = fruits.stream()
    .collect(Collectors.joining(", "));
// "elma, armut, kiraz"
```

`Collectors.joining(CharSequence delimiter)`, her eleman arasına verilen ayırıcıyı yerleştirir ama baştaki ve sondaki elemanlara fazladan bir ayırıcı eklemez; bu, elle yazılan ve son elemandan sonra fazladan virgül kalan döngülerin aksine otomatik olarak doğru davranır. Parametresiz `joining()` çağrısı ayırıcı olmadan sadece elemanları art arda ekler.

## Önek ve sonek ile birlikte kullanım

```java
String bracketed = fruits.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// "[elma, armut, kiraz]"

String empty = Stream.<String>empty()
    .collect(Collectors.joining(", ", "[", "]"));
// "[]" -- boş stream'de bile önek/sonek korunur
```

Üç parametreli aşırı yükleme `delimiter`, `prefix` ve `suffix` alır; önek ve sonek stream boş olsa bile her zaman sonuca eklenir, çünkü `toString()` çağırılan bir koleksiyonun çıktısını taklit etmek (`[a, b, c]` gibi) bu collector'ın tipik kullanım amaçlarından biridir.

## Dikkat edilmesi gerekenler

- `Collectors.joining` yalnızca `CharSequence` (yani `String`) stream'leri üzerinde çalışır; `Stream<Integer>` gibi başka tiplerde önce `.map(Object::toString)` ile dönüştürme yapılmalıdır, aksi halde derleme hatası alınır.
- Elemanlardan biri `null` ise `joining` bir `NullPointerException` fırlatır; bu durumda `map(String::valueOf)` veya `Objects.toString` ile önceden güvenli hale getirme gerekir.
- `StringJoiner` sınıfı `joining`'in arkasındaki mekanizmadır ve stream dışında, döngü içinde manuel ekleme yapılacaksa doğrudan kullanılabilir.

## Denemek için

```
jshell> import java.util.stream.*
jshell> import java.util.List
jshell> List.of("bir", "iki", "üç").stream().collect(Collectors.joining(" - ", "<", ">"))
```

Çıktıda `"<bir - iki - üç>"` değerini görürsünüz; ayırıcının elemanlar arasında, önek/sonek'in ise sadece baş ve sonda bir kez göründüğüne dikkat edin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#joining(java.lang.CharSequence,java.lang.CharSequence,java.lang.CharSequence)]
