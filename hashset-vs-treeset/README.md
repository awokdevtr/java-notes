# hashset-vs-treeset

Benzersiz elemanları tutmak için `Set` arayüzünü seçmek kolaydır, ama hangi implementasyonu seçeceğiniz sıralama garantisi ve performans arasında gerçek bir takas yaratır. `HashSet` elemanları hiçbir belirli sırada tutmazken `TreeSet` her zaman sıralı tutar; bu farkın altında yatan iç veri yapısı, basit bir "hangisi daha hızlı" sorusunun yanlış soru olduğunu gösterir.

## Sıralama garantisi ve iç yapı farkı

```java
Set<Integer> hashSet = new HashSet<>(List.of(5, 3, 8, 1));
Set<Integer> treeSet = new TreeSet<>(List.of(5, 3, 8, 1));

System.out.println(hashSet); // örn: [1, 3, 5, 8] veya farklı bir sıra, garanti yok
System.out.println(treeSet); // her zaman: [1, 3, 5, 8]
```

`HashSet`, elemanları bir hash tablosunda tutar; `add`, `contains` ve `remove` ortalama `O(1)` sürede çalışır ama iterasyon sırası `hashCode()` dağılımına bağlıdır ve garanti edilmez. `TreeSet` ise elemanları kırmızı-siyah ağaçta (`red-black tree`) tutar; bu yüzden `add`/`contains`/`remove` `O(log n)` sürer ama eleman her zaman doğal sıralamaya (`Comparable`) göre sıralı kalır. Doğal sıralaması olmayan bir sınıfın nesnesini `Comparator` vermeden `TreeSet`'e eklemeye çalışırsanız `ClassCastException` alırsınız.

## TreeSet'te özel sıralama

```java
Set<String> byLength = new TreeSet<>(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()));
byLength.addAll(List.of("elma", "armut", "üzüm", "kiraz"));
System.out.println(byLength); // uzunluğa göre artan, eşitlikte alfabetik
```

`TreeSet` yapıcısına bir `Comparator` vererek doğal sıralamayı değiştirebilirsiniz. Önemli nokta: `TreeSet` eşsizliği `equals()` ile değil, verilen `Comparator`'ın (veya `compareTo`'nun) `0` döndürüp döndürmediğiyle belirler; yani `compareTo`/`compare` sonucu `0` olan iki farklı nesne, `equals()` `false` dönse bile kümede yalnızca birinin kalmasına yol açar.

## Dikkat edilmesi gerekenler

- `HashSet` tek bir `null` elemana izin verir; `TreeSet` ise doğal sıralama kullanıldığında `null` eklemeye çalışınca `NullPointerException` fırlatır, çünkü `null.compareTo(...)` çağrılamaz.
- Sık iterasyon gerektiren ama ekleme sırasının da önemli olduğu durumlar için `LinkedHashSet` üçüncü bir seçenektir: `HashSet` gibi `O(1)` performans sağlar, ayrıca her elemanı ekleme sırasına göre tutan bir bağlı liste de taşır.
- Sadece benzersizlik yetiyorsa ve sıralama gerekmiyorsa `TreeSet`'in `O(log n)` maliyetini ödemeye değmez; `HashSet` tercih edin.

## Denemek için

```
jshell> var ts = new java.util.TreeSet<Integer>(java.util.List.of(9, 2, 5, 1))
jshell> ts
```

Çıktının her zaman `[1, 2, 5, 9]` sıralı geldiğini, aynı denemeyi `HashSet` ile tekrarladığınızda ise sıranın elemanlara ve JVM'e göre değişebildiğini göreceksiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeSet.html]
