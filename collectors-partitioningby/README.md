# Collectors.partitioningBy

`groupingBy` bir stream'i key'e göre istediğin kadar gruba ayırır, ama elemanları sadece "doğru" ve "yanlış" olarak iki kümeye bölmek istediğinde fazla iş yapar: bir `Predicate` için bile `Map<Boolean, List<T>>` değil `Map<Object, List<T>>` tipinde, potansiyel olarak eksik anahtarlı bir sonuç üretir. `Collectors.partitioningBy`, tam olarak bu ikili bölme senaryosu için var: sonuç her zaman tam iki anahtarlı (`true`/`false`) bir `Map` olur, hangi dalın boş olduğuna bakılmaksızın.

## İkili bölme

```java
List<Integer> sayilar = List.of(3, 8, 15, 22, 7, 40, 11);

Map<Boolean, List<Integer>> bolunmus = sayilar.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));

System.out.println(bolunmus.get(true));   // [8, 22, 40]
System.out.println(bolunmus.get(false));  // [3, 15, 7, 11]
```

`partitioningBy` tek bir `Predicate` alır ve her elemanı bu `predicate`'in sonucuna göre `true` veya `false` anahtarlı listeye yazar. Dönen `Map`'te her iki anahtar da garanti olarak bulunur; eşleşen eleman olmasa bile `bolunmus.get(true)` boş bir liste döner, `null` değil. Bu, `groupingBy` ile elle yazılmış bir `Predicate` grubunun aksine `containsKey` kontrolüne gerek bırakmaz.

## Downstream collector ile özetleme

```java
Map<Boolean, Long> sayilarSayisi = sayilar.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0, Collectors.counting()));

System.out.println(sayilarSayisi.get(true));   // 3
System.out.println(sayilarSayisi.get(false));  // 4
```

`groupingBy` gibi `partitioningBy` de iki parametreli bir aşırı yüklemeye sahiptir: ikinci argüman olarak bir downstream `Collector` alıp her iki dalın elemanlarını ayrı ayrı bu collector'dan geçirir. Burada `Collectors.counting()` ile her gruptaki eleman sayısı hesaplanıyor, ama `summingInt`, `mapping` veya `toList()` gibi herhangi bir collector aynı şekilde kullanılabilir.

## Ne zaman işe yarar

- Bölme kriteri gerçekten ikili olduğunda (geçti/kaldı, aktif/pasif, çift/tek) `groupingBy(predicate)` yerine tercih edilmeli; dönen tipin `Map<Boolean, ...>` olması niyeti kod okuyana da açıkça gösterir.
- Üçten fazla kategori gerektiğinde `partitioningBy` uygun değildir; `Predicate` sadece iki dal üretebilir, bu durumda `groupingBy` kullanılmalıdır.
- Döndürülen `Map`'in sırası garanti değildir ama her zaman tam olarak iki giriş (`true`, `false`) içerir; bu yüzden `getOrDefault` yerine doğrudan `get` güvenle kullanılabilir.

## Denemek için

```
jshell> import java.util.stream.*; import java.util.*
jshell> var sonuc = Stream.of(1, 2, 3, 4, 5, 6).collect(Collectors.partitioningBy(n -> n > 3))
jshell> sonuc.get(true)
jshell> sonuc.get(false)
```

`sonuc` değişkenini yazdırdığınızda `{false=[1, 2, 3], true=[4, 5, 6]}` çıktısını, yani her iki anahtarın da her koşulda dolu bir `Map`'te yer aldığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#partitioningBy(java.util.function.Predicate)]
