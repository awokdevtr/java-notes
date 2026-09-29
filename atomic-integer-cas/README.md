# AtomicInteger ve compare-and-swap

`count++` gibi bir bileşik işlem aslında oku-değiştir-yaz üç adımından oluşur; birden fazla thread bunu aynı anda çalıştırdığında adımlar iç içe geçip bir artış kaybolabilir. `synchronized` bunu kilitle çözer ama thread'i bloklar. `java.util.concurrent.atomic.AtomicInteger`, donanım seviyesindeki compare-and-swap (CAS) komutunu kullanarak aynı garantiyi kilitlemeden verir.

## Kilitsiz sayaç

```java
AtomicInteger counter = new AtomicInteger(0);

Runnable increment = () -> counter.incrementAndGet();

Thread t1 = new Thread(increment);
Thread t2 = new Thread(increment);
t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(counter.get()); // her zaman 2
```

`incrementAndGet()` içeride bir döngüde mevcut değeri okur, bir fazlasını hesaplar ve `compareAndSet(beklenen, yeni)` ile belleğe yazmayı dener; başka bir thread arada değeri değiştirmişse `compareAndSet` başarısız olur ve döngü güncel değerle yeniden dener. Bu retry-until-success mekanizması hiçbir thread'i askıya almadan atomikliği sağlar; kilitlenme (deadlock) riski de yoktur çünkü bekleyen thread yok.

## compareAndSet ile koşullu güncelleme

```java
AtomicInteger version = new AtomicInteger(1);

boolean updated = version.compareAndSet(1, 2);
System.out.println(updated);              // true, 1 ise 2 yapıldı
System.out.println(version.get());        // 2

boolean stale = version.compareAndSet(1, 3);
System.out.println(stale);                // false, mevcut değer artık 1 değil
```

`compareAndSet(expected, newValue)`, değeri yalnızca `expected` ile eşleşiyorsa değiştirir ve sonucu `boolean` döner; eşleşmezse hiçbir şey yapmaz. Bu, optimistic locking'in temelidir: thread'ler birbirini bloklamadan çalışır, çakışma anında yalnızca kaybeden taraf yeniden dener.

## Ne zaman işe yarar

- Basit sayaç, id üreteci veya bayrak gibi tek bir değişken üzerinde yüksek eşzamanlı güncelleme gerektiğinde `synchronized`'a göre çok daha az bekleme (contention) yaratır.
- Birden fazla alanı birlikte tutarlı güncellemek gerekiyorsa (örneğin iki alanı birlikte değiştirme) CAS yetmez, çünkü tek bir referans/alan üzerinde atomiktir — bu durumda `synchronized` veya `AtomicReference` ile immutable bir taşıyıcı nesne tercih edilmelidir.
- Çok yüksek çekişme (contention) altında CAS'ın sürekli başarısız olup yeniden denemesi, kilide göre daha fazla CPU harcayabilir; bu senaryoda `LongAdder` gibi contention'a özel sınıflar daha uygundur.

## Denemek için

```
jshell> var c = new java.util.concurrent.atomic.AtomicInteger(10)
jshell> c.incrementAndGet()
jshell> c.compareAndSet(11, 100)
jshell> c.get()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html]
