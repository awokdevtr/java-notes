# synchronized anahtar kelimesi

Birden fazla thread aynı değişkeni okuyup değiştirdiğinde, `count++` gibi görünüşte tek adımlık işlemler aslında oku-değiştir-yaz şeklinde üç ayrı adımdır; iki thread bu adımları iç içe geçirirse bir güncelleme kaybolur. `synchronized`, bir kod bloğunu ya da metodu bir seferde yalnızca tek bir thread'in çalıştırabilmesini garanti eden karşılıklı dışlama (mutual exclusion) mekanizmasıdır.

## Metot ve blok senkronizasyonu

```java
class Counter {
    private int count = 0;

    synchronized void increment() {
        count++;
    }

    int get() {
        synchronized (this) {
            return count;
        }
    }
}
```

`synchronized` bir metoda eklendiğinde, çağrının çalışabilmesi için thread'in `this` nesnesinin monitor (intrinsic lock) kilidini alması gerekir; kilit boştaysa thread devam eder, doluysa serbest kalana kadar bloke olur. `synchronized (this) { ... }` bloğu aynı kilidi daha dar bir kapsamda kullanmayı sağlar. Kilit yalnızca karşılıklı dışlamayı değil, `volatile` gibi görünürlüğü de garanti eder: kilidi bırakan thread'in yazdığı değerler, aynı kilidi sonradan alan thread'e görünür olur.

## static synchronized ve sınıf kilidi

```java
class IdGenerator {
    private static int nextId = 0;

    static synchronized int next() {
        return nextId++;
    }
}
```

Bir `static` metot `synchronized` olduğunda kilit, herhangi bir örnek değil, `IdGenerator.class` nesnesinin kendisidir. Bu yüzden aynı sınıfın örnek metotlarındaki `synchronized (this)` ile static metotlardaki `synchronized` birbirinden bağımsız iki farklı kilittir; biri diğerini bloke etmez, karıştırıldığında yarış koşulu (race condition) sessizce devam eder.

## Dikkat edilmesi gerekenler

- Kilit yeniden girebilir (reentrant): bir thread zaten sahip olduğu kilidi tekrar isteyebilir, örneğin bir `synchronized` metot içinden aynı nesnenin başka bir `synchronized` metodunu çağırmak deadlock oluşturmaz.
- Birden fazla nesneyi farklı sırada kilitleyen kod parçaları arasında çapraz bekleme oluşursa deadlock riski doğar.
- `volatile` yalnızca görünürlük sağlar; `synchronized` hem görünürlüğü hem de bileşik işlemlerin atomikliğini sağlar, ama kilit bekleme maliyeti nedeniyle daha pahalıdır.
- İnce ayarlı kontrol gerektiğinde `java.util.concurrent.atomic.AtomicInteger` veya `java.util.concurrent.locks.ReentrantLock` gibi sınıflar `synchronized`'a göre daha esnek alternatiflerdir.

## Denemek için

```
jshell> class Counter { int c = 0; synchronized void inc() { c++; } }
jshell> Counter counter = new Counter()
jshell> Runnable task = () -> { for (int i = 0; i < 100000; i++) counter.inc(); }
jshell> Thread t1 = new Thread(task); Thread t2 = new Thread(task)
jshell> t1.start(); t2.start(); t1.join(); t2.join(); counter.c
```

`inc()` metodundan `synchronized` kaldırılıp aynı deney tekrarlanırsa, sonucun 200000'den küçük çıkması yüksek ihtimaldir; kilit olmadan iki thread'in `c++` adımları iç içe geçer ve bazı artışlar kaybolur.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.1]
