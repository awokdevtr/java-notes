# BigDecimal vs double

`double`, IEEE 754 ikili kayan nokta formatını kullanır ve `0.1` gibi ondalık sayıları tam olarak temsil edemez; her aritmetik işlemde küçük yuvarlama hataları birikir. Para tutarları gibi kuruş hassasiyeti gereken hesaplamalarda bu hata görünmez şekilde büyür ve toplamlar tutmaz. `BigDecimal`, sayıyı ölçeklenmiş bir tamsayı (`unscaledValue` ve `scale`) olarak tuttuğu için ondalık kesirleri kayıpsız temsil eder.

## Kayan nokta hatasının görünür hâli

```java
double a = 1.03;
double b = 0.42;
System.out.println(a - b);   // 0.6100000000000001

BigDecimal x = new BigDecimal("1.03");
BigDecimal y = new BigDecimal("0.42");
System.out.println(x.subtract(y));   // 0.61
```

`1.03` ve `0.42` ikili kesir olarak tam gösterilemediğinden, `double` çıkarması matematiksel olarak yanlış ama `double` aritmetiğine göre "doğru" bir sonuç üretir: `0.6100000000000001`. `BigDecimal` ise ölçeklenmiş tamsayılar üzerinde çıkarma yaptığı için sonucu kesin olarak `0.61` verir. `BigDecimal` oluştururken `String` constructor'ı kullanmak kritiktir; `new BigDecimal(0.42)` çağrısı `double` değerinin zaten bozulmuş ikili temsilini miras alır.

## Bölme ve yuvarlama stratejisi belirtme zorunluluğu

```java
BigDecimal total = new BigDecimal("100.00");
BigDecimal parts = new BigDecimal("3");
BigDecimal share = total.divide(parts, 2, RoundingMode.HALF_UP);
// share = 33.34
```

`double` bölmesi sonucu her zaman bir kayan nokta değeri üretirken, `BigDecimal.divide` sonucu sonlu ondalık basamakla ifade edilemezse (`100/3` gibi) `ArithmeticException` fırlatır; bu yüzden ölçek (`scale`) ve `RoundingMode` açıkça belirtilmelidir. Bu zorunluluk, para hesaplamalarında yuvarlama kuralının kodda gizli kalmasını değil, açıkça seçilmesini sağlar.

## Ne zaman işe yarar

- Para birimi, faiz, vergi gibi kuruş/sent hassasiyetinin yasal veya muhasebesel olarak önemli olduğu her hesaplama `BigDecimal` gerektirir.
- Bilimsel hesaplama, grafik, sensör verisi gibi küçük hata payının kabul edilebilir olduğu ve performansın öncelikli olduğu durumlarda `double` yeterlidir; `BigDecimal` nesne tabanlı olduğu için `double`'dan belirgin şekilde yavaştır.
- `equals()` ölçeğe duyarlıdır: `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` `false` döner; sayısal eşitlik için `compareTo() == 0` kullanılmalıdır.

## Denemek için

```
jshell> import java.math.BigDecimal
jshell> import java.math.RoundingMode
jshell> System.out.println(0.1 + 0.2)
jshell> new BigDecimal("0.1").add(new BigDecimal("0.2"))
```

İlk satır `0.30000000000000004` basar; ikinci satır `BigDecimal` ile tam olarak `0.3` döner.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigDecimal.html]
