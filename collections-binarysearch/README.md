# collections-binarysearch

`Arrays.binarySearch()` dizilerde ikili arama yapar, ama elinizdeki veri bir `List` ise (özellikle `LinkedList` gibi rastgele erişimi `O(n)` olan bir yapıda) diziye çevirmeden aynı performans kazancını elde etmek gerekir. `Collections.binarySearch()` bu ihtiyacı karşılar, fakat aynı sessiz ön koşulu taşır: liste önceden **sıralı** olmalı, ve bu metot bunu kendisi doğrulamaz.

## Sıralı listede arama ve ekleme noktası

```java
List<Integer> values = new ArrayList<>(List.of(1, 3, 5, 7, 9, 11));

int found = Collections.binarySearch(values, 7);
System.out.println(found); // 3 (index)

int missing = Collections.binarySearch(values, 6);
System.out.println(missing); // -4 -> insertionPoint = -(-4) - 1 = 3
```

`Arrays.binarySearch()` ile aynı sözleşmeyi paylaşır: eleman bulunursa indeksini, bulunamazsa elemanın sıralı yapıda hangi noktaya ekleneceğini kodlayan negatif bir değer döner (`-(insertionPoint) - 1`). Fark, bu algoritmanın liste `RandomAccess` arayüzünü uygulayıp uygulamamasına göre iki ayrı stratejiye sahip olmasıdır.

## RandomAccess olmayan listelerde strateji değişimi

```java
List<Integer> linked = new LinkedList<>(List.of(2, 4, 6, 8, 10));

int result = Collections.binarySearch(linked, 8);
System.out.println(result); // 3 -- ama O(log n) indeks atlaması yerine iteratör taraması kullanılır
```

`List`, `RandomAccess` arayüzünü uyguluyorsa (`ArrayList` gibi) metot klasik indeks tabanlı ikili aramayı kullanır. Uygulamıyorsa (`LinkedList` gibi) `java.util.Collections` içindeki uygulama otomatik olarak `Iterator` tabanlı bir aramaya geçer; bu da karşılaştırma sayısını `O(log n)`'de tutsa da, iteratörle ilerleme maliyeti yüzünden gerçek kazancı `ArrayList`'teki kadar büyük olmaz.

## Dikkat edilmesi gerekenler

- Liste, aramadan önce doğal sıraya göre veya arama sırasında verilen `Comparator` ile **aynı** `Comparator`'a göre sıralanmış olmalı; aksi halde sonuç tanımsızdır ve `ArrayIndexOutOfBoundsException` gibi görünür bir hata fırlatılmaz.
- Listede eşit anahtarlı birden fazla eleman varsa, dönen indeksin hangisine ait olduğu garanti edilmez.
- `LinkedList` gibi `RandomAccess` olmayan büyük listelerde bu metodu çağırmak, baştan diziye çevirip `Arrays.binarySearch()` kullanmaktan daha yavaş kalabilir; veri sık aranacaksa uygun veri yapısını seçmek daha önemlidir.

## Denemek için

```
jshell> var list = new java.util.ArrayList<>(java.util.List.of(10, 20, 30, 40))
jshell> java.util.Collections.binarySearch(list, 25)
jshell> java.util.Collections.binarySearch(list, 30)
```

İlk çağrı negatif bir ekleme noktası, ikincisi ise doğrudan indeks döndürür; bu ikisini karşılaştırmak sözleşmenin iki yüzünü birlikte gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html#binarySearch(java.util.List,java.lang.Object)]
