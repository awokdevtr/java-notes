# Collectors.groupingBy ile eleman gruplama

Bir listeyi belirli bir özelliğe göre gruplara ayırmak, döngü ve `Map` ile elle yazıldığında birkaç satırlık tekrar eden kod gerektirir: anahtar yoksa yeni liste oluştur, varsa mevcut listeye ekle. `Collectors.groupingBy` bu deseni tek bir terminal operasyona indirger ve `Stream` API'sinin `collect` metoduyla doğrudan kullanılır.

## Temel gruplama: anahtar başına liste

```java
record Person(String name, String city) {}

List<Person> people = List.of(
    new Person("Ayşe", "İzmir"),
    new Person("Mehmet", "Ankara"),
    new Person("Zeynep", "İzmir")
);

Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::city));

// {Ankara=[Person[name=Mehmet, city=Ankara]], İzmir=[Person[name=Ayşe, city=İzmir], Person[name=Zeynep, city=İzmir]]}
```

`groupingBy` içteki her eleman için `classifier` fonksiyonunu (`Person::city`) çalıştırır ve dönen değeri anahtar olarak kullanır. Sonuç varsayılan olarak `HashMap<K, List<V>>` tipindedir; her anahtar için eşleşen elemanlar sırayla bir `List`'e toplanır. `classifier` saf bir fonksiyon olmalıdır, çünkü stream paralel çalıştığında aynı anahtar üzerinde eşzamanlı birleştirme yapılabilir.

## İkinci parametre: downstream collector ile sonucu dönüştürme

```java
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(Person::city, Collectors.counting()));
// {Ankara=1, İzmir=2}

Map<String, Set<String>> namesByCity = people.stream()
    .collect(Collectors.groupingBy(Person::city, Collectors.mapping(Person::name, Collectors.toSet())));
```

İkinci argüman olarak verilen `downstream` collector, her grubun `List<Person>` yerine nasıl özetleneceğini belirler. `counting()` grup boyutunu sayar, `mapping()` ise önce her elemanı dönüştürüp sonra başka bir collector'a (`toSet()`, `joining()` gibi) besler. Bu iki parametreli form, gruplama ile toplama/özetleme mantığını tek bir `collect` çağrısında birleştirir.

## Dikkat edilmesi gerekenler

- Anahtar olarak `null` döndüren bir `classifier`, `HashMap` tabanlı sonuçta `NullPointerException` yerine `null` anahtarını kabul eder çünkü `HashMap` `null` anahtara izin verir; ancak `groupingBy` bunu garanti etmez, `classifier` `null` döndürmemelidir.
- Üç parametreli aşırı yüklenmiş sürüm, `mapFactory` ile sonucun `HashMap` yerine `TreeMap` gibi başka bir `Map` implementasyonu olmasını sağlar; anahtar sıralaması önemliyse bu kullanılmalıdır.
- Paralel stream'lerde `groupingBy` yerine eşzamanlılık için `Collectors.groupingByConcurrent` tercih edilebilir, ancak bu durumda dönen `Map` sıra garantisi vermez.

## Denemek için

```
jshell> import java.util.stream.*
jshell> var words = List.of("elma", "armut", "erik", "ayva")
jshell> words.stream().collect(Collectors.groupingBy(String::length))
```

Çıktıda anahtarların kelime uzunluğu, değerlerin ise aynı uzunluktaki kelimelerin listesi olduğunu gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html]
