# StringBuilder vs String birleştirme

Java'da `String` değişmezdir (immutable); `+` ile iki string birleştirildiğinde mevcut nesneler değişmez, bellekte yeni bir `String` nesnesi oluşturulur. Bir döngü içinde tekrar tekrar `+` kullanmak, her adımda atılacak ara nesneler yaratır ve bu durum büyük veri kümelerinde performansı ciddi şekilde etkiler. `StringBuilder`, değiştirilebilir bir karakter dizisi tutarak bu maliyeti ortadan kaldırır.

## Döngüde `+` kullanmanın maliyeti

```java
String result = "";
for (int i = 0; i < 10_000; i++) {
    result = result + i; // her adımda yeni String nesnesi
}
```

Derleyici tek bir `+` ifadesini arka planda `StringBuilder`'a çevirse de, bu dönüşüm yalnızca tek bir ifade için geçerlidir; döngü gövdesindeki her iterasyon kendi `StringBuilder` örneğini oluşturup `toString()` çağırır. Sonuç olarak 10.000 iterasyonda 10.000 ara `String` nesnesi üretilir ve bunların çoğu hemen çöp toplayıcının (garbage collector) işine kalır.

## StringBuilder ile tek nesne üzerinde biriktirme

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10_000; i++) {
    sb.append(i); // aynı iç buffer üzerinde büyür
}
String result = sb.toString();
```

`StringBuilder.append` çağrıları, kapasitesi yetmediğinde otomatik olarak büyüyen tek bir iç `char[]`/`byte[]` buffer üzerinde çalışır; döngü boyunca yalnızca bir nesne mutasyona uğrar. `toString()` yalnızca döngü bittikten sonra bir kez çağrılarak nihai `String` üretilir. Bu yaklaşım, ara nesne sayısını sabit tutar ve büyük birleştirmelerde belirgin şekilde daha hızlıdır.

## Ne zaman ise yarar

- Sabit sayıda, tek bir ifadedeki `+` birleştirmeleri için derleyici zaten optimize eder; `StringBuilder`'ı elle yazmak gerekmez.
- Döngü, koşullu ekleme veya çok sayıda parça birleştirme söz konusu olduğunda `StringBuilder` tercih edilmelidir.
- Thread-safe bir alternatif gerekiyorsa `StringBuffer` kullanılabilir; ancak senkronizasyon ek yük getirdiğinden tek thread'li kodda `StringBuilder` yeterlidir.

## Denemek için

```
jshell> var sb = new StringBuilder()
jshell> for (int i = 0; i < 5; i++) sb.append(i).append(",")
jshell> sb.toString()
```

`append` çağrılarının aynı `sb` nesnesi üzerinde biriktiğini ve her adımda yeni bir nesne oluşmadığını gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html]
