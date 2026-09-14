# priority-queue

Bir işlem sırasını her zaman "en küçük" ya da "en yüksek öncelikli" elemana göre yönetmek gerektiğinde, elemanları her seferinde elle sıralamak hem gereksiz hem de maliyetlidir. `ArrayList`'i her ekleme sonrası `sort()` ile sıralamak `O(n log n)` işe mal olur. `PriorityQueue`, iç yapısında bir `binary heap` tutarak ekleme ve en öncelikli elemanı çıkarma işlemlerini `O(log n)`'e indirger; sıralama işini tamamlamadan yalnızca sıradaki elemanı garanti eder.

## Temel kullanım

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(1);
pq.offer(3);

while (!pq.isEmpty()) {
    System.out.println(pq.poll());
}
// Çıktı: 1, 3, 5
```

Varsayılan olarak `PriorityQueue`, elemanların doğal sırasını (`Comparable.compareTo`) kullanarak en küçük elemanı köke yerleştirir; `poll()` her çağrıldığında bu kök elemanı döndürür ve heap'i yeniden düzenler. `offer()` ve `poll()` çağrıları `O(log n)` sürer, ancak kuyruğun `iterator()`'ü ile gezinmek elemanları sıralı sırada döndürmez, çünkü heap yalnızca kökte minimum garantisi verir.

## Comparator ile özel öncelik sırası

```java
record Task(String name, int priority) {}

PriorityQueue<Task> tasks = new PriorityQueue<>(
    Comparator.comparingInt(Task::priority).reversed()
);
tasks.offer(new Task("deploy", 1));
tasks.offer(new Task("hotfix", 10));
tasks.offer(new Task("cleanup", 3));

System.out.println(tasks.poll().name()); // hotfix
```

Kurucuya bir `Comparator` verildiğinde doğal sıra devre dışı kalır ve heap bu karşılaştırıcıya göre kurulur. `reversed()` kullanarak "en yüksek öncelik önce" gibi bir davranış elde etmek, elemanları `Comparable` yapmaya zorlamadan esnek sıralama kuralları tanımlamayı sağlar.

## Dikkat edilmesi gerekenler

- `PriorityQueue` thread-safe değildir; çoklu iş parçacığından erişim için `PriorityBlockingQueue` kullanılmalıdır.
- `null` eleman eklenemez, `NullPointerException` fırlatılır.
- `remove(Object)` ile rastgele bir elemanı kaldırmak `O(n)` sürer, çünkü heap yalnızca kökte hızlı erişim sağlar.
- `toString()` veya `iterator()` çıktısı elemanları öncelik sırasında **göstermez**; sıralı sonuç için tekrar tekrar `poll()` çağırmak gerekir.

## Denemek için

```
jshell> var pq = new java.util.PriorityQueue<Integer>(java.util.Collections.reverseOrder())
jshell> pq.addAll(java.util.List.of(4, 9, 1, 7))
jshell> while (!pq.isEmpty()) System.out.println(pq.poll())
```

`Collections.reverseOrder()` ile kurulan kuyruk, `poll()` çağrıldığında elemanları büyükten küçüğe (`9, 7, 4, 1`) döndürür; bu da `Comparator` değişiminin heap'in kök seçimini nasıl etkilediğini gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PriorityQueue.html]
