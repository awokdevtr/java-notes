# Arrays.sort İç Yapısı

`Arrays.sort()` tek bir metot gibi görünür ama altında iki farklı algoritma çalışır ve hangisinin seçileceği elemanların türüne bağlıdır. Bu ayrımı bilmemek, "neden ilkel dizi sıralaması stabil değil ama nesne dizisi sıralaması stabil" gibi sorulara yanlış cevaplar üretmenize yol açabilir; stabilite ve performans garantileri birbirine karıştırılırsa, özellikle eşit anahtarlı nesneleri sıralarken beklenmeyen sıra değişimleri ortaya çıkar.

## İlkel diziler: Dual-Pivot Quicksort

```java
int[] numbers = {5, 3, 5, 1, 4, 3};
Arrays.sort(numbers);
System.out.println(Arrays.toString(numbers));
```

`int[]`, `long[]`, `double[]` gibi ilkel tür dizilerinde `Arrays.sort()`, ortalama durumda `O(n log n)` çalışan bir Dual-Pivot Quicksort kullanır. Bu algoritma `stabil değildir` ama ilkellerde zaten eşit değerler birbirinden ayırt edilemediği için stabilite kavramı anlamsızdır; önemli olan tek şey hız ve yer karmaşıklığıdır. Quicksort ek dizi ayırmadan yerinde (`in-place`) çalıştığı için bellek ayak izi küçüktür.

## Nesne dizileri: TimSort

```java
String[] words = {"muz", "elma", "armut", "elma"};
Arrays.sort(words);
```

`Object[]` türündeki (veya `Comparable`/`Comparator` kullanan) dizilerde ise `Arrays.sort()`, birleştirme sıralaması (`merge sort`) ile ekleme sıralamasının (`insertion sort`) bir karışımı olan TimSort'u çalıştırır. TimSort `stabildir`: `equals()` açısından eşit sayılan elemanlar, orijinal dizideki göreli sıralarını korur. Bu garanti, `Comparator.thenComparing()` ile çok alanlı sıralama yapıldığında önceki sıralama adımının sonucunun bozulmamasını sağlar; ancak TimSort birleştirme adımı için `O(n)` ek bellek kullanır, bu yüzden ilkel dizilerdeki quicksort'tan daha fazla yer harcar.

## Dikkat edilmesi gerekenler

- `Comparable` uygulamayan ya da tutarsız bir `compareTo()` yazan nesnelerle sıralama yaparken TimSort `IllegalArgumentException: Comparison method violates its general contract!` fırlatabilir; quicksort bu tutarsızlığı sessizce görmezden geçer.
- `Arrays.parallelSort()` yeterince büyük dizilerde Fork/Join tabanlı paralel bir birleştirme sıralaması kullanır; küçük dizilerde sıralı sürüme otomatik olarak düşer.
- `List.sort()` de aynı TimSort temelini paylaşır, dolayısıyla `Collections.sort()` ile `Arrays.sort(Object[])` arasındaki stabilite garantisi tutarlıdır.

## Denemek için

```
jshell> int[] a = {3, 1, 2}; java.util.Arrays.sort(a); System.out.println(java.util.Arrays.toString(a))
jshell> Integer[] b = {3, 1, 2}; java.util.Arrays.sort(b); System.out.println(java.util.Arrays.toString(b))
```

Her ikisi de `[1, 2, 3]` döndürür ama ilk satır ilkel Dual-Pivot Quicksort'u, ikinci satır kutulanmış `Integer[]` üzerinden TimSort'u çalıştırır; `javap` ile `Arrays.sort` metotlarının farklı overload'larına bakarsanız bu ayrımı imzalarda da görebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#sort(int%5B%5D)]
