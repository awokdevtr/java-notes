# Collectors.toUnmodifiableList/Set/Map ile Değiştirilemez Koleksiyon Üretme

`Stream.collect(Collectors.toList())` döndürdüğü liste, hiçbir yerde belgelenmeyen bir varsayıma dayanır: genellikle değiştirilebilir bir `ArrayList`'tir ve çağıran kod bunu fark etmeden üzerine `add`/`remove` çağırabilir. Elde edilen koleksiyonun gerçekten değişmez olmasını garanti etmek için `Collectors.toUnmodifiableList`, `toUnmodifiableSet` ve `toUnmodifiableMap` kullanılır; bunlar stream'in `collect` adımında doğrudan değişmez bir yapı üretir.

## Temel kullanım: değiştirilemez liste üretme

```java
List<String> isimler = Stream.of("Ayşe", "Mehmet", "Zeynep")
    .filter(isim -> isim.length() > 5)
    .collect(Collectors.toUnmodifiableList());

isimler.add("Ali"); // UnsupportedOperationException
```

`Collectors.toUnmodifiableList()`, `Collectors.toList()`'in aksine sonucu baştan `List.of(...)` benzeri gerçek bir değişmez yapıda döndürür; `Collections.unmodifiableList` gibi arkada değiştirilebilir bir listeye bağlı bir "görünüm" değildir, çünkü koleksiyon stream tarafından içeride inşa edilip hiçbir dış referansa sızdırılmadan paketlenir. Bu yüzden `isimler` üzerinde yapılan her değiştirme denemesi anında `UnsupportedOperationException` fırlatır.

## Set ve Map varyantları

```java
Set<Integer> uzunluklar = Stream.of("a", "bb", "ccc")
    .collect(Collectors.toUnmodifiableSet());

Map<String, Integer> uzunlukMap = Stream.of("a", "bb", "ccc")
    .collect(Collectors.toUnmodifiableMap(s -> s, String::length));

uzunlukMap.put("dd", 2); // UnsupportedOperationException
```

`toUnmodifiableSet` ve `toUnmodifiableMap` aynı garantiyi sırasıyla `Set` ve `Map` için verir. `toUnmodifiableMap`, çakışan anahtarlarda `Collectors.toMap` ile aynı davranışı sürdürür: iki eleman aynı anahtara eşlenirse varsayılan olarak `IllegalStateException` fırlatır; çözüm için üç parametreli aşırı yükleme bir `mergeFunction` kabul eder.

## Dikkat edilmesi gerekenler

- Hiçbiri `null` eleman ya da `null` anahtar/değer kabul etmez; bir `null` üretimi `collect` sırasında `NullPointerException` ile sonuçlanır.
- Java 16 ile gelen `Stream.toList()` terminal metodu da değişmez bir liste döndürür ve çoğu durumda `collect(Collectors.toUnmodifiableList())`'ten daha kısa yazılır; ikisi aynı garantiyi verir ama `Stream.toList()` ayrı bir `Collectors` çağrısı gerektirmez.
- Elde edilen koleksiyon üzerinde `removeIf`, `sort` gibi yapıyı değiştiren metotlar da aynı istisnayı fırlatır; sadece `add`/`remove`/`put` sınırlı değildir.

## Denemek için

```
jshell> import java.util.stream.*
jshell> import java.util.*
jshell> var liste = Stream.of("a","b","c").collect(Collectors.toUnmodifiableList())
jshell> liste.add("d")
```

Son satır `UnsupportedOperationException` fırlatır; `liste` değişkeninin `Collectors.toList()` ile üretilmiş olsaydı bu çağrının sorunsuz çalışacağını karşılaştırarak görebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toUnmodifiableList()]
