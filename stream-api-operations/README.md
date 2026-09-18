# Stream API'de ara ve terminal operasyonlar

`Stream` üzerindeki metotlar iki kategoriye ayrılır: **ara (intermediate)** operasyonlar yeni bir stream döndürür ve tembeldir (lazy), **terminal** operasyonlar ise stream'i tüketip bir sonuç ya da yan etki üretir. Bu ayrımı anlamadan yazılan kod, hiçbir şey yapmıyormuş gibi görünen ya da beklenmedik anda çalışan filtre/map zincirlerine yol açar.

## Ara operasyonlar terminal çağrılana kadar çalışmaz

```java
Stream<String> stream = Stream.of("a", "bb", "ccc")
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.length() > 1;
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    });

System.out.println("Terminal öncesi hiçbir şey basılmadı");
stream.forEach(System.out::println);
```

`filter` ve `map` çağrıları anında çalışmaz; sadece stream'in yapacağı işlemleri tanımlayan bir zincir kurarlar. Gerçek işlem, `forEach` gibi bir **terminal** operasyon çağrıldığında, her eleman için tüm ara adımlardan tek tek geçirilerek (element-by-element) başlar. Bu tembel değerlendirme sayesinde stream, gereksiz elemanları işlemeden kısa devre yapabilir — örneğin `findFirst()` eşleşen ilk elemanı bulur bulmaz kalanları hiç dokunmadan bırakır.

## Bir stream yalnızca bir kez tüketilebilir

```java
Stream<Integer> numbers = Stream.of(1, 2, 3);
long count = numbers.count(); // terminal operasyon, stream artık kapandı

numbers.forEach(System.out::println); // IllegalStateException
```

Terminal bir operasyon çalıştıktan sonra stream "tüketilmiş" sayılır ve üzerinde başka bir işlem çağırmak `IllegalStateException: stream has already been operated upon or closed` fırlatır. Her seferinde yeni bir işlem zinciri gerekiyorsa stream yeniden oluşturulmalıdır (örneğin kaynağı bir `Supplier` içine sarıp her seferinde `.get()` ile taze bir stream almak yaygın bir çözümdür).

## Dikkat edilmesi gerekenler

- `filter`, `map`, `sorted`, `distinct`, `peek` gibi metotlar ara operasyondur; `forEach`, `collect`, `reduce`, `count`, `anyMatch`, `findFirst` terminal operasyondur.
- Yan etkisi olmayan, saf fonksiyonlar kullanmak gerekir; `peek` sadece hata ayıklama amaçlı kullanılmalı, iş mantığı için değil.
- Sonsuz stream'ler (`Stream.iterate`, `Stream.generate`) ancak `limit` gibi bir ara operasyonla sınırlandıktan sonra terminal operasyonla güvenle tüketilebilir.

## Denemek için

```
jshell> var s = java.util.stream.Stream.of(1,2,3,4).peek(x -> System.out.println("gorulen: " + x))
jshell> s.filter(x -> x % 2 == 0).findFirst()
```

`peek` çıktısının yalnızca `findFirst` çağrılana kadar, ve yalnızca gerekli olan elemanlar için basıldığını gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html]
