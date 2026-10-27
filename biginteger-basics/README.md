# BigInteger temelleri

`int` 32 bit, `long` 64 bit sınırına sahiptir ve bu sınırı aşan bir aritmetik işlem sessizce taşar (`overflow`); sonuç matematiksel olarak yanlış olsa bile exception fırlamaz. Kriptografi, kombinatorik hesaplamalar veya rastgele büyüklükte tamsayı gerektiren algoritmalarda bu sınır yetersiz kalır. `java.math.BigInteger`, sayıyı iç bir `int[]` dizisinde keyfi uzunlukta tutarak bellek sınırına kadar kesin tamsayı aritmetiği sağlar.

## Taşmanın görünür hâli ve BigInteger çözümü

```java
long factorial20 = 1;
for (int i = 1; i <= 20; i++) factorial20 *= i;
System.out.println(factorial20);               // 2432902008176640000 (henüz doğru)

BigInteger factorial25 = BigInteger.ONE;
for (int i = 1; i <= 25; i++) {
    factorial25 = factorial25.multiply(BigInteger.valueOf(i));
}
System.out.println(factorial25);                // 15511210043330985984000000
```

`20!` hâlâ `long` sınırları içinde kalır, ama `25!` gibi bir değer `long`'un 64 bitlik sınırını (`Long.MAX_VALUE`) aşar ve `long` ile hesaplanırsa sessizce yanlış bir sonuç üretir. `BigInteger` her çarpma işleminde sonucu yeni bir nesnede büyüterek sınır olmadan doğru değeri tutar. `BigInteger` **immutable**'dır: `multiply`, `add`, `subtract` gibi her metot orijinali değiştirmeden yeni bir `BigInteger` döndürür, bu yüzden sonucu her zaman yeni değişkene atamak gerekir.

## Bölme, mod ve karşılaştırma

```java
BigInteger a = new BigInteger("123456789012345678901234567890");
BigInteger b = BigInteger.valueOf(7);

BigInteger[] result = a.divideAndRemainder(b);
System.out.println(result[0] + " kalan " + result[1]);

System.out.println(a.compareTo(BigInteger.ZERO) > 0);   // true
```

`divideAndRemainder`, bölüm ve kalanı tek çağrıda `BigInteger[2]` olarak döndürür ve `ArithmeticException` sadece sıfıra bölmede fırlar — ondalık sonuç gerektiren bir durum söz konusu olmadığından `BigDecimal`'deki gibi yuvarlama stratejisi belirtmeye ihtiyaç yoktur. Eşitlik ve sıralama için `==` veya `equals()` yerine `compareTo()` tercih edilmelidir; `equals()` teknik olarak doğru çalışır ama `compareTo() == 0` kod tabanında tutarlılık açısından daha yaygındır.

## Ne zaman işe yarar

- `long`'un 64 bit sınırını aşan kesin tamsayı hesapları: büyük faktöriyel, kombinasyon, Fibonacci dizisinin ileri terimleri.
- Kriptografik algoritmalar (`RSA` gibi) büyük asal sayılar ve modüler üs alma (`modPow`) üzerine kuruludur; bunlar doğrudan `BigInteger` ile ifade edilir.
- Performans kritik, sınırları bilinen sıradan sayaç ve döngü değişkenleri için `BigInteger` gereksiz nesne tahsisi ve kat kat daha yavaş aritmetik getirir; bu durumlarda `int`/`long` yeterlidir.

## Denemek için

```
jshell> import java.math.BigInteger
jshell> BigInteger.valueOf(2).pow(100)
jshell> BigInteger.probablePrime(64, new java.util.Random())
```

İlk satır `2^100`'ü tam olarak basar (`long` ile bu hesap taşardı); ikinci satır her çalıştırmada 64 bitlik olasılıksal bir asal sayı üretir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigInteger.html]
