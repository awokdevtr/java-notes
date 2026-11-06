# parallel-stream-pitfalls

`.parallelStream()` ya da `.parallel()` çağırmak, bir stream'i otomatik olarak daha hızlı hale getirmez; çoğu zaman tam tersi olur. Alt seviyede `ForkJoinPool.commonPool()` üzerinde çalışan bu mekanizma, küçük koleksiyonlarda thread başlatma ve parçalama (`split`) maliyetinin işin kendisinden daha ağır basmasına, paylaşılan mutable durumla birlikte kullanıldığında da sessiz veri yarışlarına yol açabilir.

## Paylaşılan durum tuzağı

```java
List<Integer> sayilar = IntStream.rangeClosed(1, 10_000).boxed().toList();
List<Integer> sonuc = new ArrayList<>();

sayilar.parallelStream().forEach(sonuc::add);
System.out.println(sonuc.size());
```

`ArrayList` thread-safe olmadığı için birden fazla thread aynı anda `add()` çağırdığında `ArrayIndexOutOfBoundsException` fırlayabilir ya da boyut 10.000'den küçük çıkabilir; hata her çalıştırmada aynı şekilde kendini göstermeyebilir. Doğru kullanım, `Collectors.toList()` gibi thread-safe biriktirme sağlayan bir `collect()` çağrısıdır, çünkü her alt görev kendi ara sonucunu tutar ve birleştirme (`combine`) aşaması merkezi olarak yapılır.

## Küçük veri setinde ek maliyet

```java
long toplam = IntStream.rangeClosed(1, 1000)
        .parallel()
        .sum();
```

Bu boyuttaki bir veri üzerinde paralel çalışma, işi parçalara bölmek, her parça için görev oluşturmak ve sonuçları `commonPool()`'daki thread'ler arasında birleştirmek için sıralı koleksiyonlara göre genellikle daha fazla zaman harcar. Paralelliğin kazanç sağlaması için eleman sayısının yüksek olması ve her elemandaki işin (`CPU-bound`) göreli olarak pahalı olması gerekir; aksi halde `sequential()` daha hızlıdır.

## Dikkat edilmesi gerekenler

- `parallel()`, sıralamayı garanti eden `findFirst()` gibi operasyonlarda `findAny()`'e kıyasla ek senkronizasyon maliyeti getirir.
- Tüm paralel stream'ler aynı `ForkJoinPool.commonPool()`'u paylaşır; uygulamanın başka bir yerinde bu havuzu dolduran uzun süren bir görev varsa, ilgisiz bir paralel stream de bloklanabilir.
- Kaynak koleksiyonun `Spliterator` desteği zayızsa (örneğin `LinkedList`), parçalama verimsiz çalışır ve paralellik beklenen kazancı sağlamaz; `ArrayList` ya da dizi tabanlı kaynaklar daha iyi bölünür.

## Denemek için

```
jshell> import java.util.stream.IntStream
jshell> long t0 = System.nanoTime(); IntStream.rangeClosed(1, 1000).parallel().sum(); System.out.println((System.nanoTime() - t0) / 1000)
jshell> long t1 = System.nanoTime(); IntStream.rangeClosed(1, 1000).sequential().sum(); System.out.println((System.nanoTime() - t1) / 1000)
```

İlk ölçümün (parallel) genellikle ikinciden (sequential) daha yüksek mikrosaniye değeri verdiğini görürsünüz; fark, küçük veri setlerinde paralelleştirme yükünün net kazancı nasıl geride bıraktığını gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html#Parallelism]
