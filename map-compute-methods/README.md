# Map.computeIfAbsent, merge ve getOrDefault

`Map`'te sayaç tutmak veya anahtara göre liste biriktirmek için klasik yol `containsKey` kontrolü yapıp sonra `get`/`put` çağırmaktır; bu hem iki-üç satır tekrar kod yazdırır hem de tek thread'li olmayan senaryolarda kontrol ile yazma arasında yarış durumu açar. Java 8 ile `Map` arayüzüne eklenen `getOrDefault`, `computeIfAbsent` ve `merge` default metotları bu deseni tek bir atomik çağrıya indirir.

## computeIfAbsent ile tembel oluşturma

```java
Map<String, List<Integer>> groups = new HashMap<>();
groups.computeIfAbsent("even", k -> new ArrayList<>()).add(2);
groups.computeIfAbsent("even", k -> new ArrayList<>()).add(4);

System.out.println(groups);
```

`computeIfAbsent`, anahtar haritada yoksa `mappingFunction`'ı çalıştırıp sonucu haritaya koyar ve döndürür; anahtar zaten varsa fonksiyonu hiç çağırmadan mevcut değeri döndürür. Bu sayede "yoksa oluştur, varsa getir" mantığı tek satıra iner ve `ConcurrentHashMap` üzerinde atomik olarak çalışır. Dikkat: `mappingFunction` haritayı değiştirirse (örneğin aynı haritaya `put` çağırırsa) `ConcurrentModificationException` veya tanımsız davranış riski vardır; fonksiyon yalnızca yeni değeri üretmelidir.

## merge ile sayaç ve birleştirme

```java
Map<String, Integer> wordCount = new HashMap<>();
for (String word : List.of("a", "b", "a", "c", "a")) {
    wordCount.merge(word, 1, Integer::sum);
}

System.out.println(wordCount);
```

`merge(key, value, remappingFunction)`, anahtar yoksa doğrudan `value`'yu koyar; anahtar varsa mevcut değer ile `value`'yu `remappingFunction`'a verip sonucu yazar. `remappingFunction` `null` döndürürse anahtar haritadan tamamen silinir. Kelime sayma, toplam biriktirme gibi "varsa güncelle, yoksa başlat" işlemleri `merge` ile `computeIfAbsent`'ten bile daha kısa yazılır.

## Dikkat edilmesi gerekenler

- `getOrDefault`, haritaya hiçbir şey yazmaz; sadece `get` çağrısının `null` döndürdüğü durumda bir varsayılan değer verir, `computeIfAbsent` ise eksik anahtarı haritaya ekler.
- `HashMap` üzerinde bu metotlar atomik değildir; gerçek thread-safety için `ConcurrentHashMap` kullanılmalı ve `mappingFunction` içinde haritaya yeniden erişilmemelidir.
- `merge`'ün `remappingFunction`'ı `null` döndürdüğünde anahtarı sildiğini unutmak, beklenmeyen eleman kayıplarına yol açabilir.

## Denemek için

```
jshell> var m = new java.util.HashMap<String, Integer>()
jshell> m.merge("a", 1, Integer::sum)
jshell> m.merge("a", 1, Integer::sum)
jshell> m.computeIfAbsent("b", k -> 0)
```

İlk `merge` çağrısı "a" anahtarını `1` ile başlatır, ikincisi mevcut değere ekleyerek `2` yapar; `computeIfAbsent` ise "b" anahtarı olmadığı için onu `0` ile haritaya ekler.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html#computeIfAbsent(K,java.util.function.Function)]
