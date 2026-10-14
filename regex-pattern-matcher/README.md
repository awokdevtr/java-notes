# Pattern ve Matcher

`String.matches()` ya da `String.replaceAll()` kullanışlı görünür, ama her çağrıda verdiğiniz regex dizesini baştan derler. Aynı deseni bir döngüde yüzlerce kez kontrol ederseniz bu derleme maliyeti katlanarak birikir. `java.util.regex` paketindeki `Pattern` ve `Matcher` sınıfları, derleme adımını eşleştirme adımından ayırarak hem performans hem de daha zengin bir eşleştirme API'si sunar.

## Derleme ve eşleştirme

```java
Pattern pattern = Pattern.compile("\\d{3}-\\d{4}");
Matcher matcher = pattern.matcher("Telefon: 555-1234");

if (matcher.find()) {
    System.out.println(matcher.group());
}
```

`Pattern.compile()` bir regex'i bir kez derleyip yeniden kullanılabilir, `immutable` bir `Pattern` nesnesi üretir. `matcher()` metodu bu desenle belirli bir karakter dizisi üzerinde durum tutan (`stateful`) bir `Matcher` döner. `matches()` tüm girdinin desenle eşleşmesini isterken, `find()` girdi içinde desenin bir sonraki geçtiği yeri arar ve art arda çağrıldığında aramaya kaldığı yerden devam eder.

## Yakalama grupları

```java
Pattern pattern = Pattern.compile("(\\w+)@(\\w+)\\.com");
Matcher matcher = pattern.matcher("destek@example.com");

if (matcher.matches()) {
    System.out.println(matcher.group(1));
    System.out.println(matcher.group(2));
}
```

Parantez içine alınan her alt desen bir yakalama grubu oluşturur ve `group(0)` tüm eşleşmeyi, `group(1)`, `group(2)` sırayla alt grupları verir. Java 20 itibarıyla `(?<ad>...)` sözdizimiyle gruplara isim verip `group("ad")` ile erişmek de mümkündür; bu, grup sırasına bağımlı kırılgan kodu önler.

## Dikkat edilmesi gerekenler

- Aynı `Pattern` nesnesi thread-safe'tir ve paylaşılabilir, ama ürettiği `Matcher` nesneleri durum tuttuğu için thread-safe değildir.
- Bir döngüde sabit bir regex'i tekrar tekrar eşleştiriyorsanız `Pattern`'i döngü dışında bir kez derleyin; `String.matches()` her çağrıda yeniden derler.
- Geriye izleme (`backtracking`) gerektiren karmaşık desenler, kontrolsüz girdilerle üstel zaman karmaşıklığına (`ReDoS`) yol açabilir.

## Denemek için

```
jshell> import java.util.regex.*
jshell> var m = Pattern.compile("[aeiou]").matcher("merhaba")
jshell> while (m.find()) System.out.println(m.start() + ": " + m.group())
```

Her bulunan sesli harfin konumunu ve kendisini yazdığını, `find()`'ın imleci bir sonraki eşleşmeye taşıdığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html]
