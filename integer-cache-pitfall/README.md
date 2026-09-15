# Integer Cache Tuzağı

`Integer i1 = 100; Integer i2 = 100;` yazınca `i1 == i2` doğru sonucu verir, ama `Integer i1 = 200; Integer i2 = 200;` yazınca aynı karşılaştırma yanlış döner. İkisi de aynı `autoboxing` işlemini yapıyor gibi görünür, fakat `==` operatörünün referans karşılaştırdığı unutulduğunda bu davranış çoğu geliştiriciyi şaşırtır ve üretimde sessizce yanlış sonuçlar üreten karşılaştırma hatalarına yol açar.

## Cache neden var, nasıl çalışır

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true

Integer c = 128;
Integer d = 128;
System.out.println(c == d); // false
```

`Integer.valueOf(int)` çağrısı, `-128` ile `127` arasındaki değerler için önceden oluşturulmuş bir nesne havuzundan referans döndürür; bu aralık dışına çıkıldığında her seferinde yeni bir `Integer` nesnesi oluşturulur. Derleyici, `Integer a = 127;` gibi ilkel-den-kutulanmış (`autoboxing`) atamaları arka planda `Integer.valueOf(127)` çağrısına çevirir, doğrudan `new Integer(127)` yapmaz. Bu yüzden aynı küçük değer iki değişkene atandığında `==` aynı nesneyi işaret ettiği için `true` döner, ama havuz dışındaki değerlerde her atama farklı bir nesne ürettiği için `==` `false` sonucu verir. Aynı mekanizma `Byte`, `Short`, `Long` ve `Character` (0-127 arası) için de geçerlidir.

## Doğru karşılaştırma

```java
Integer c = 200;
Integer d = 200;
System.out.println(c.equals(d));        // true
System.out.println(Integer.compare(c, d) == 0); // true
```

Kutulanmış tiplerde değer karşılaştırması her zaman `equals()` ya da ilkel türe indirgeyip (`c.intValue() == d.intValue()`) yapılmalıdır; `==` sadece iki referansın aynı nesneyi gösterip göstermediğini söyler, cache aralığı bu davranışı değer aralığına bağlı hale getirir.

## Dikkat edilmesi gerekenler

- Cache aralığı `-XX:AutoBoxCacheMax` JVM parametresiyle büyütülebilir ama küçültülemez; bu aralığa güvenerek kod yazmak taşınabilir değildir.
- `new Integer(127)` (Java 9'dan itibaren `deprecated`) her zaman yeni nesne oluşturur ve cache'i tamamen atlar; bu yüzden `Integer.valueOf()` veya autoboxing tercih edilmelidir.
- `HashMap<Integer, V>` gibi koleksiyonlarda anahtar karşılaştırması `equals()`/`hashCode()` üzerinden yapıldığından cache bu tür kullanımları etkilemez; risk sadece doğrudan `==` kullanımında ortaya çıkar.

## Denemek için

```
jshell> Integer a = 127, b = 127; System.out.println(a == b)
jshell> Integer c = 128, d = 128; System.out.println(c == d)
```

İlk satır `true`, ikinci satır `false` yazdırır; aradaki tek fark değerin cache aralığının (`-128`..`127`) içinde mi dışında mı olduğudur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Integer.html#valueOf(int)]
