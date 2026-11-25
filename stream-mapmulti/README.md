# Stream.mapMulti ile tek elemandan çoklu üretim

`flatMap`, bir elemanı başka bir `Stream`'e dönüştürüp onları birleştirir, ama her eleman için ayrı bir `Stream` nesnesi oluşturmak zorundasınızdır — küçük veya sıfır sayıda çıktı üretilen durumlarda bu gereksiz bir nesne yaratma maliyetine yol açar. `Stream.mapMulti`, `Stream` oluşturmak yerine bir `Consumer` üzerinden doğrudan çıktı eleman(lar)ını yayınlamanıza izin vererek bu maliyeti ortadan kaldırır.

## Tek elemandan sıfır, bir veya birden fazla değer üretme

```java
record Siparis(String musteri, List<String> urunler) {}

List<Siparis> siparisler = List.of(
    new Siparis("Ayşe", List.of("kalem", "defter")),
    new Siparis("Can", List.of()),
    new Siparis("Ela", List.of("silgi"))
);

List<String> tumUrunler = siparisler.stream()
    .<String>mapMulti((siparis, consumer) -> {
        for (String urun : siparis.urunler()) {
            consumer.accept(urun);
        }
    })
    .toList();
```

`mapMulti`'ye verilen `BiConsumer<T, Consumer<R>>`, her giriş elemanı için çağrılır ve içindeki `consumer.accept(...)` kaç kez çağrılırsa o kadar eleman çıktı stream'ine eklenir. Hiç çağrılmazsa o eleman için hiçbir şey üretilmez, yani `filter` ve `flatMap`'in birleşik etkisini tek bir ara operasyonla elde edersiniz. Jenerik tip çıkarımı genellikle elemanın türünü belirleyemediği için `<R>mapMulti(...)` şeklinde açık tip belirtmek gerekir.

## mapMulti'nin flatMap'e göre avantajı

```java
IntStream.rangeClosed(1, 5)
    .mapToObj(n -> n)
    .mapMulti((n, consumer) -> {
        if (n % 2 == 0) consumer.accept(n);
        if (n > 3) consumer.accept(n * 10);
    })
    .forEach(System.out::println);
```

`flatMap` burada her eleman için küçük bir `Stream.of(...)` nesnesi kurup o nesneyi stream pipeline'ına eklemek zorunda kalırdı; `mapMulti` ise ara nesne oluşturmadan doğrudan `consumer.accept` ile değer basar. Özellikle az sayıda (`0` veya `1`) çıktı üreten yoğun döngülerde bu, ölçülebilir bir performans kazancı sağlar.

## Dikkat edilmesi gerekenler

- İlkel tip stream'leri için `mapMultiToInt`, `mapMultiToLong` ve `mapMultiToDouble` varyantları vardır; boxing'den kaçınmak için onlar tercih edilmelidir.
- `consumer`'ı stream kapandıktan sonra veya başka bir thread'den çağırmak tanımsız davranıştır; çağrı sadece `mapMulti`'ye verilen `BiConsumer` içinde, senkron olarak yapılmalıdır.
- Çıktı sayısı sabit ve `1` ise `map`, `flatMap`'in bir `Stream` oluşturmaya değecek kadar karmaşık olduğu durumlarda `flatMap` daha okunaklı kalabilir; `mapMulti` asıl faydasını değişken sayıda (`0..n`) çıktı üretiminde gösterir.

## Denemek için

```
jshell> import java.util.stream.*;
jshell> List.of("a", "bb", "ccc").stream().<Character>mapMulti((s, c) -> s.chars().forEach(ch -> c.accept((char) ch))).toList()
jshell> IntStream.rangeClosed(1, 10).mapMultiToInt((n, c) -> { if (n % 3 == 0) c.accept(n); }).sum()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#mapMulti(java.util.function.BiConsumer)]
