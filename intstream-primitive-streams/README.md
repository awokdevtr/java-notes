# IntStream ve primitive stream'ler

`Stream<Integer>` kullanmak masum görünür ama her eleman için bir `Integer` nesnesi anlamına gelir: `int`'ten kutuya girer (`autoboxing`), işlenir, sonra tekrar kutudan çıkar (`unboxing`). Milyonlarca elemanlı bir toplama işleminde bu, gereksiz nesne oluşturma ve `heap` baskısı demektir. Java, sayısal veriler için kutulama hiç yapmayan ayrı stream türleri sunar: `IntStream`, `LongStream`, `DoubleStream`.

## Boxing olmadan toplama

```java
int[] values = {3, 1, 4, 1, 5, 9, 2, 6};

int sum = java.util.Arrays.stream(values)
    .filter(n -> n % 2 == 0)
    .sum();

System.out.println(sum); // 12
```

`Arrays.stream(int[])` doğrudan bir `IntStream` döndürür; elemanlar hiçbir noktada `Integer`'a sarılmaz. `IntStream` zaten `sum()`, `average()`, `max()`, `min()` gibi terminal operasyonları hazır sunar — genel `Stream<T>` üzerinde bu toplamları almak için `Collectors.summingInt` gibi bir ara adıma ihtiyaç duyarsınız. `filter` ve `map` burada `IntPredicate`/`IntUnaryOperator` alır, `Function<Integer, R>` değil.

## Stream<Integer> ile IntStream arasında geçiş

```java
java.util.List<String> words = java.util.List.of("elma", "armut", "kiraz");

int totalLength = words.stream()
    .mapToInt(String::length)
    .sum();
```

`mapToInt`, `mapToLong`, `mapToDouble` bir nesne stream'ini ilgili primitive stream'e çevirir; tersi yönde `boxed()` metodu primitive stream'i tekrar `Stream<Integer>`'a sarar. Bir hesaplama tamamen sayısalsa zincir boyunca primitive stream'de kalmak, her adımda kutulama/kutudan çıkarma maliyetinden kurtarır ve `IntSummaryStatistics` gibi özetleme araçlarını terminal operasyon olarak kullanmanıza izin verir.

## Dikkat edilmesi gerekenler

- `IntStream`, `Stream<Integer>`'ın alt tipi değildir; ikisi birbirinden bağımsız arayüzlerdir, aralarında yalnızca `mapToX`/`boxed` ile dönüşüm yapılır.
- `IntStream.range(a, b)` üst sınırı hariç tutar, `IntStream.rangeClosed(a, b)` dahil eder — döngü sınırlarını birbirine karıştırmak klasik bir off-by-one hatasıdır.
- `IntStream` içindeki `average()` ve `sum()` gibi metotlar `int` taşmasına (`overflow`) karşı korumasızdır; çok büyük toplamlarda `LongStream` veya `summaryStatistics()` tercih edilmelidir.

## Denemek için

```
jshell> var stats = java.util.stream.IntStream.rangeClosed(1, 10).summaryStatistics()
jshell> stats.getSum()
jshell> stats.getAverage()
```

Tek bir terminal çağrıyla toplam, ortalama, min ve max değerlerinin aynı anda hesaplandığını, ayrı ayrı stream gezmeye gerek kalmadığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/IntStream.html]
