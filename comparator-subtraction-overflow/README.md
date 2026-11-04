# Comparator Çıkarma Tuzağı (Subtraction Overflow)

Bir `Comparator` ya da `compareTo` yazarken iki `int` değeri birbirinden çıkarıp sonucu doğrudan döndürmek yaygın bir kısayoldur: `return a - b;`. Kısa ve okunaklı görünür, çoğu test verisinde de doğru çalışır. Ancak `a` ve `b` değerleri `int` aralığının uçlarına (`Integer.MIN_VALUE`/`MAX_VALUE`) yaklaştığında çıkarma işlemi taşar (`overflow`), işaret değişir ve sıralama sessizce bozulur — istisna fırlamadığı için hata fark edilmeden üretime sızabilir.

## Taşan çıkarma örneği

```java
Comparator<Integer> hatali = (a, b) -> a - b;

int sonuc = hatali.compare(Integer.MIN_VALUE, 1);
System.out.println(sonuc); // taşma nedeniyle pozitif bir değer, oysa MIN_VALUE < 1
```

`Integer.MIN_VALUE - 1` matematiksel olarak `int` sınırlarının dışına çıkar; JVM bunu `wraparound` ile sarar ve sonuç `Integer.MAX_VALUE` olur — yani pozitif. Çağıran kod bunu "`a` daha büyük" diye yorumlar, hâlbuki `a` değeri `b`'den çok daha küçüktür. `TreeMap`, `TreeSet` veya `Collections.sort` gibi sıralı yapılarda bu, elemanların kaybolmasına veya yanlış sıraya girmesine yol açar.

## Güvenli alternatif

```java
Comparator<Integer> dogru1 = Integer::compare;
Comparator<Integer> dogru2 = (a, b) -> Integer.compare(a, b);
Comparator<Calisan> dogru3 = Comparator.comparingInt(c -> c.maas);
```

`Integer.compare(x, y)` taşma riski olmadan işaret karşılaştırması yapar ve `-1`, `0`, `1` döndürür; iç implementasyonu çıkarma değil doğrudan `<`/`>` karşılaştırmasıdır. `Comparator.comparingInt` da aynı güvenli yolu kullanır, bu yüzden `int` alanlara göre sıralarken çıkarma yazmak yerine bu metotlardan biri tercih edilmelidir.

## Dikkat edilmesi gerekenler

- Aynı tuzak `long` çıkarmanın sonucunu `int`'e sıkıştırırken de ortaya çıkar (`(int) (longA - longB)`); taşma burada hem büyüklükten hem de tipten kaynaklanabilir.
- Hata genellikle yalnızca uç değerlerle (çok büyük/küçük sayılar, `Integer.MIN_VALUE`) karşılaşıldığında görülür, bu yüzden normal test verileriyle fark edilmeyebilir.
- `double`/`float` alanlar için çıkarma taşmaz ama `NaN` karşılaştırmasında benzer bir "sessiz hata" riski vardır; orada da `Double.compare` kullanmak daha güvenlidir.

## Denemek için

```
jshell> Comparator<Integer> hatali = (a, b) -> a - b
jshell> hatali.compare(Integer.MIN_VALUE, 1)
jshell> Integer.compare(Integer.MIN_VALUE, 1)
```

İlk çağrı taşma nedeniyle pozitif bir sayı (`2147483647`) döndürür, ikinci çağrı ise doğru şekilde `-1` verir; aynı iki değer için iki farklı "kim büyük" sonucu, çıkarma tabanlı karşılaştırmanın neden güvenilmez olduğunu gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Integer.html#compare(int,int)]
