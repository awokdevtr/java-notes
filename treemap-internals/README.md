# treemap-internals

`HashMap` anahtarları gezinirken rastgele bir sırada gelir ve ekleme/arama ortalama `O(1)`'dir, ama hiçbir sıralama garantisi vermez. Anahtarların her zaman sıralı tutulmasını ve en kötü durumda da öngörülebilir bir performansın gerekmesini istediğinizde `TreeMap` devreye girer; içeride kırmızı-siyah ağaç (red-black tree) kullanarak `get`, `put` ve `remove` işlemlerini en kötü durumda da `O(log n)`'de garanti eder.

## Sıralama kaynağı: doğal sıra ve Comparator

```java
TreeMap<String, Integer> naturalOrder = new TreeMap<>();
naturalOrder.put("muz", 3);
naturalOrder.put("elma", 1);
naturalOrder.put("kiraz", 2);
System.out.println(naturalOrder.keySet()); // [elma, kiraz, muz]

TreeMap<String, Integer> byLength = new TreeMap<>(Comparator.comparingInt(String::length));
byLength.put("muz", 3);
byLength.put("elma", 1);
byLength.put("kiraz", 2);
System.out.println(byLength.keySet()); // [muz, elma, kiraz] (uzunluğa göre, 3-4-5)
```

Kurucuya bir `Comparator` verilmezse anahtarların `compareTo` metoduyla tanımlı doğal sırası kullanılır; bu yüzden anahtar sınıfı `Comparable` uygulamıyorsa ve `Comparator` da verilmemişse `ClassCastException` fırlar. Bir `Comparator` verildiğinde ise `equals`/`hashCode` tamamen devre dışı kalır: iki anahtar `compare` sonucu `0` döndürüyorsa `TreeMap` onları aynı kabul eder, ikincisi birinciyi değiştirir.

## Dengeleme maliyeti ve ara düğüm kısıtlaması

```java
TreeMap<Integer, String> tree = new TreeMap<>();
for (int i = 1; i <= 1_000_000; i++) {
    tree.put(i, "v" + i);
}
System.out.println(tree.get(500_000)); // v500000, derinlik ~20 düğüm
```

Ağaç her ekleme ve silmede kendini yeniden dengeler, böylece derinlik her zaman `O(log n)` sınırında kalır; bir milyon elemanlı bir `TreeMap`'te en uzun arama yolu bile yaklaşık 20 karşılaştırma gerektirir. Bu dengeleme, `HashMap`'in ortalama `O(1)` erişimine göre sabit bir ek maliyet getirir, dolayısıyla sadece sıralı gezinme veya `firstKey`/`lastKey` gibi sıra bağımlı işlemler gerekiyorsa `TreeMap` tercih edilmelidir.

## Dikkat edilmesi gerekenler

- `TreeMap` eşzamanlı değildir; birden fazla iş parçacığı aynı anda değiştirirse `ConcurrentModificationException` veya tutarsız durum oluşabilir, bunun için `ConcurrentSkipListMap` kullanılmalıdır.
- `null` anahtar kabul edilmez (`compareTo` çağrısı `NullPointerException` fırlatır), ama `Comparator` null'u özel olarak işliyorsa bu kısıtlama aşılabilir.
- `keySet()`, `values()` ve `entrySet()` sırayı her zaman artan anahtar sırasına göre yansıtır; tersten gezinmek için `descendingMap()` veya `descendingKeySet()` kullanılabilir.

## Denemek için

```
jshell> var t = new java.util.TreeMap<String,Integer>()
jshell> t.put("muz", 3); t.put("elma", 1); t.put("kiraz", 2)
jshell> t.firstKey()
jshell> t.keySet()
```

`firstKey()` çağrısı `"elma"` döner; `keySet()` ise `[elma, kiraz, muz]` sırasını verir, çünkü `TreeMap` anahtarları her zaman sıralı tutar.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeMap.html]
