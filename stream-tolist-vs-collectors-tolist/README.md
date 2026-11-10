# Stream.toList() ile Collectors.toList() farkı

Java 16'dan önce bir `Stream`'i listeye dönüştürmenin tek yolu `collect(Collectors.toList())` idi. Java 16 ile gelen `Stream.toList()` kısayolu aynı işi görür gibi dursa da, döndürdüğü listenin değiştirilebilirliği ve iç implementasyon garantileri bakımından `Collectors.toList()`'ten farklıdır; bu fark elle fark edilmesi zor, sessiz bir `UnsupportedOperationException` kaynağıdır.

## Değiştirilebilirlik farkı

```java
List<String> names = List.of("Ali", "Veli", "Ayşe");

List<String> viaCollectors = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
viaCollectors.add("Zeynep"); // çalışır

List<String> viaToList = names.stream()
    .filter(n -> n.length() > 3)
    .toList();
viaToList.add("Zeynep"); // UnsupportedOperationException
```

`Collectors.toList()` dokümantasyonda dönen listenin tipi, değiştirilebilirliği, serileştirilebilirliği veya thread-safe'liği konusunda garanti vermez; pratikte `ArrayList` döner ve bu yüzden eleman eklenebilir. `Stream.toList()` ise bilinçli olarak değiştirilemez bir liste döndürür, bu yüzden `add`, `remove` veya `set` çağrıları her zaman istisna fırlatır.

## null eleman farkı

```java
List<String> withNull = Arrays.asList("a", null, "b");

withNull.stream().collect(Collectors.toList()); // null içerebilir, sorun yok
withNull.stream().toList();                     // stream null içeriyorsa NullPointerException fırlatabilir
```

`Stream.toList()` içeride `List.of` benzeri bir değişmez liste kullandığından, dönüştürülen elemanlar arasında `null` varsa `NullPointerException` fırlatır. `Collectors.toList()` ise `null` elemanlara izin veren sıradan bir `ArrayList` ürettiği için bu kısıtlama yoktur.

## Ne zaman ise yarar

- Sonucun daha fazla değiştirilmeyeceği, salt okunur bir anlık görüntü olarak kullanılacağı durumlarda `Stream.toList()` daha az kod ve daha güvenli bir değişmezlik garantisi sağlar.
- Sonuca eleman eklenip çıkarılacaksa veya `null` elemanlar bekleniyorsa `collect(Collectors.toList())` (ya da açıkça `Collectors.toCollection(ArrayList::new)`) tercih edilmelidir.

## Denemek için

```
jshell> var l1 = java.util.List.of(1, 2, 3).stream().map(x -> x * 2).toList()
jshell> l1.add(10)
```

Fırlatılan `UnsupportedOperationException`'ı gözlemleyin; aynı denemeyi `.collect(java.util.stream.Collectors.toList())` ile tekrarladığınızda `add`'in sorunsuz çalıştığını görün.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#toList()]
