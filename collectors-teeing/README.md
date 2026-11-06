# Collectors.teeing ile tek geçişte iki sonucu birleştirme

Bir stream üzerinde aynı anda iki farklı özet çıkarmak istendiğinde (örneğin ortalama ve sayım, veya minimum ve maksimum) elle yazılan çözüm genelde stream'i iki kere işlemek veya bir ara `List`'e toplayıp sonra iki ayrı hesap yapmak şeklinde olur. `Collectors.teeing`, aynı elemanları tek geçişte iki collector'a paralel olarak besler ve sonuçlarını bir `BiFunction` ile birleştirir.

## Temel kullanım: ortalama ve sayımı birlikte hesaplama

```java
record Sale(String product, double amount) {}

List<Sale> sales = List.of(
    new Sale("kalem", 12.5),
    new Sale("defter", 30.0),
    new Sale("silgi", 5.0)
);

record Summary(double average, long count) {}

Summary summary = sales.stream()
    .collect(Collectors.teeing(
        Collectors.averagingDouble(Sale::amount),
        Collectors.counting(),
        Summary::new
    ));
// Summary[average=15.833333333333334, count=3]
```

`teeing` üç parametre alır: iki `downstream` collector ve bunların sonuçlarını birleştiren bir `merger` fonksiyonu. Stream her elemanı aynı anda hem birinci hem ikinci collector'a aktarır; iki collector birbirinden habersiz, bağımsız olarak kendi akümülasyonunu yapar. Terminal operasyon tamamlandığında `merger`, iki ayrı sonucu alıp tek bir nesneye dönüştürür.

## İkinci örnek: min ve max'ı birlikte bulma

```java
List<Integer> numbers = List.of(7, 2, 9, 4, 1);

var minMax = numbers.stream()
    .collect(Collectors.teeing(
        Collectors.minBy(Integer::compareTo),
        Collectors.maxBy(Integer::compareTo),
        (min, max) -> min.get() + ".." + max.get()
    ));
// "1..9"
```

Burada her iki `downstream` collector `Optional` döndürdüğü için `merger` içinde `get()` çağrılmadan önce boş stream ihtimaline karşı dikkatli olunmalıdır; aksi halde boş bir stream üzerinde `NoSuchElementException` fırlatılır.

## Dikkat edilmesi gerekenler

- `teeing`, iki collector'ı paralel çalıştırmaz; aynı eleman sırasıyla ikisine de beslenir, bu yüzden performans kazancı yoktur — asıl kazanç stream'i iki kez işlemek zorunda kalmamaktır.
- İki `downstream` collector'dan biri stream'i tüketen bir yan etki (örneğin paylaşılan bir koleksiyona ekleme) içeriyorsa, bu akümülasyonun ikisi arasında bağımsız kalması gerektiği unutulmamalıdır.
- Üç collector'ı birleştirmek gerekiyorsa `teeing` doğrudan desteklemez; bu durumda iç içe `teeing` çağrıları veya ayrı bir özet `record` ile manuel toplama tercih edilmelidir.

## Denemek için

```
jshell> import java.util.stream.*
jshell> var nums = List.of(3, 1, 4, 1, 5, 9)
jshell> nums.stream().collect(Collectors.teeing(Collectors.summingInt(n -> n), Collectors.counting(), (sum, count) -> (double) sum / count))
```

Çıktıda toplam ve eleman sayısından hesaplanan ortalamayı tek bir `double` değer olarak görebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#teeing(java.util.stream.Collector,java.util.stream.Collector,java.util.function.BiFunction)]
