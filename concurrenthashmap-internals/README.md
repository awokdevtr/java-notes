# ConcurrentHashMap iç yapısı

`HashMap` birden fazla thread tarafından eşzamanlı değiştirildiğinde veri bozulmasına ve sonsuz döngülere yol açabilir, `Collections.synchronizedMap` ise tüm haritayı tek bir kilitle sarar ve eşzamanlı okuma/yazmaları seri hale getirir. `ConcurrentHashMap` bu ikisi arasında üçüncü bir yol sunar: kilitlemeyi bucket seviyesine indirerek yüksek eşzamanlılıkta çalışır.

## Bucket seviyesinde kilitleme

```java
ConcurrentHashMap<String, Integer> sayaç = new ConcurrentHashMap<>();
sayaç.put("istek", 1);
sayaç.computeIfAbsent("hata", k -> 0);
```

Java 8 öncesinde `ConcurrentHashMap` haritayı sabit sayıda segmente bölüp her segmenti ayrı kilitlerdi. Java 8 ile bu model terk edildi: artık her bucket'ın ilk düğümü `synchronized` ile kilitlenir ve `CAS` (`compare-and-swap`) tabanlı atomik işlemler kullanılır, böylece farklı bucket'lara yapılan `put` çağrıları birbirini hiç bloklamaz. `computeIfAbsent`, `merge` gibi metotlar da bu ince taneli kilitlemeyi kullanarak atomik güncellemeyi tek metot çağrısında garanti eder.

## Zayıf tutarlı okuma ve yineleme

```java
for (var girdi : sayaç.entrySet()) {
    System.out.println(girdi.getKey() + "=" + girdi.getValue());
}
```

`ConcurrentHashMap` üzerindeki `get` çağrıları ve `iterator`'lar hiç kilit almaz; `volatile` okumalarla çalışır ve `weakly consistent` davranış sergiler. Bu, yineleme sırasında haritada yapılan değişikliklerin `ConcurrentModificationException` fırlatmayacağı, ancak iteratörün o anki durumu tam olarak yansıtmayabileceği anlamına gelir — yineleme başladıktan sonra eklenen bir eleman görünebilir de görünmeyebilir de.

## Ne zaman işe yarar

- Çok sayıda thread'in aynı haritaya sık sık yazdığı ve okuduğu önbellek, sayaç veya paylaşılan durum senaryolarında.
- `size()` metodunun kesin anlık değer değil, yaklaşık bir tahmin döndürdüğünü bilerek kullanmak gerekir; kesinlik gerekiyorsa harici senkronizasyon eklenmelidir.
- `null` anahtar veya değer kabul etmez; bu, `get` çağrısının eleman yokluğu ile `null` değer arasındaki belirsizliği eşzamanlı ortamda ortadan kaldırır.

## Denemek için

```
jshell> var m = new java.util.concurrent.ConcurrentHashMap<String,Integer>(); m.put("a", 1); m.computeIfAbsent("b", k -> 2); System.out.println(m)
```

Çıktı `{a=1, b=2}` olur; `computeIfAbsent` çağrısı tek bir atomik işlemdir, aradan başka bir thread girip aynı anahtara yazmaya çalışsa da sonuç tutarlı kalır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html]
