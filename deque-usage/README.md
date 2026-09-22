# deque-usage

Bir yığın (stack) ve bir kuyruk (queue) gerektiğinde çoğu zaman iki farklı sınıfa başvurulur: eski `Stack` sınıfı `Vector`'dan türediği için senkronizasyon yükü taşır, `LinkedList`'i kuyruk gibi kullanmak ise niyeti belirsizleştirir. `Deque` (double-ended queue) arayüzü, dizinin her iki ucundan da ekleme/çıkarma yapılabilmesini tek bir sözleşme altında toplayarak hem yığın hem kuyruk davranışını aynı tipte sunar.

## Yığın (LIFO) olarak kullanım

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
System.out.println(stack.pop());  // 3
System.out.println(stack.peek()); // 2
```

`push`, `pop` ve `peek` metotları sırasıyla `addFirst`, `removeFirst` ve `peekFirst` çağrılarına eşdeğerdir; yani en son eklenen eleman ilk çıkar (`LIFO`). `ArrayDeque`, dahili olarak yeniden boyutlanabilen bir dizi kullandığı için `Stack` sınıfına göre hem daha hızlıdır hem de senkronize değildir. Java dokümantasyonu, yığın ihtiyaçlarında `Stack` yerine `ArrayDeque`'in tercih edilmesini önerir.

## Kuyruk (FIFO) olarak kullanım

```java
Deque<String> queue = new ArrayDeque<>();
queue.offer("a");
queue.offer("b");
queue.offer("c");
System.out.println(queue.poll()); // a
System.out.println(queue.peek()); // b
```

`offer`, `poll` ve `peek` burada `addLast`, `removeFirst` ve `peekFirst` çağrılarına karşılık gelir; eklenen ilk eleman ilk çıkar (`FIFO`). Aynı `ArrayDeque` örneği, sadece hangi uç metotlarının çağrıldığına bağlı olarak hem yığın hem kuyruk gibi davranabilir.

## Dikkat edilmesi gerekenler

- `ArrayDeque` `null` elemanı kabul etmez; `null` eklemeye çalışmak `NullPointerException` fırlatır. `LinkedList` bunu tolere eder, bu yüzden ikisi birbirinin yerine tam olarak geçmez.
- `addFirst`/`removeFirst`/`peekFirst` ile `addLast`/`removeLast`/`peekLast` çiftleri her zaman kullanılabilir; `push`/`pop`/`offer`/`poll` bunların üzerine kurulu kısayollardır.
- `Deque`, `Queue` arayüzünü de genişletir; bu yüzden bir `Deque` referansı hem kuyruk hem yığın API'siyle çalışabilir, ama hangi uçtan işlem yaptığınızı kod okunurluğu için net tutmak gerekir.

## Denemek için

```
jshell> Deque<Integer> d = new java.util.ArrayDeque<>();
jshell> d.push(1); d.push(2); d.offer(3);
jshell> d
```

`d`'nin içeriğini yazdırdığınızda `[2, 1, 3]` görürsünüz: `push` başa, `offer` sona ekler. Ardından `d.pop()` çağırırsanız `2` döner, `d.pollLast()` çağırırsanız `3` döner; aynı yapının iki ucundan bağımsız olarak çıkarma yapılabildiğini doğrulamış olursunuz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html]
