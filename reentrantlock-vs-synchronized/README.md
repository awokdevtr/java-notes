# ReentrantLock vs synchronized

`synchronized` anahtar kelimesi karşılıklı dışlama (mutual exclusion) için basit ve güvenli bir yoldur, ama JVM'in dahili monitor mekanizmasına bağlı olduğu için esneklik tanımaz: kilidi denemeden bekleme, zaman aşımıyla kilit alma ya da adil (fair) sıralama gibi ihtiyaçlar doğduğunda yetersiz kalır. `java.util.concurrent.locks.ReentrantLock` aynı yeniden-girebilir (reentrant) kilit semantiğini sunarken bu kontrolü programcıya bırakır.

## Temel kullanım farkı

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private int value = 0;
    private final ReentrantLock lock = new ReentrantLock();

    void increment() {
        lock.lock();
        try {
            value++;
        } finally {
            lock.unlock();
        }
    }
}
```

`synchronized` bloğunda kilidin açılması derleyici tarafından garanti edilir; `ReentrantLock` ile bu sorumluluk sana aittir. `lock()` çağrısından sonraki kod mutlaka `try` içine alınmalı ve `unlock()` `finally` bloğunda çağrılmalıdır, aksi halde bir `RuntimeException` kilidi sonsuza dek açık bırakabilir. Her iki mekanizma da aynı thread'in kilidi tekrar almasına (`reentrant`) izin verir, yani bir metot kendi kilidini tutarken aynı kilidi gerektiren başka bir metodu çağırabilir.

## Zaman aşımlı ve kesilebilir kilitleme

```java
if (lock.tryLock(500, java.util.concurrent.TimeUnit.MILLISECONDS)) {
    try {
        // kritik bölge
    } finally {
        lock.unlock();
    }
} else {
    // kilit alınamadı, deadlock'a girmeden devam et
}
```

`synchronized` bir thread'i kilit serbest kalana kadar süresiz bloke eder ve bu bekleme kesilemez (`interrupt` edilemez). `tryLock(timeout, unit)` ise belirli bir süre bekleyip vazgeçmeyi, `lockInterruptibly()` ise bekleme sırasında `InterruptedException` ile çıkabilmeyi sağlar. Bu, deadlock riskini azaltmak veya kullanıcı iptaliyle uyumlu çalışmak için kritik bir araçtır.

## Ne zaman ise yarar

`ReentrantLock` şu durumlarda tercih edilir: kilit denemesini zaman aşımına bağlamak, bekleyen thread'ler arasında adil (FIFO) sıralama istemek (`new ReentrantLock(true)`), ya da birden fazla `Condition` nesnesiyle (`newCondition()`) ince taneli bekle/uyandır mantığı kurmak gerektiğinde. Basit, tek koşullu senkronizasyon ihtiyaçlarında `synchronized` hem daha az hataya açıktır hem de JIT tarafından daha agresif optimize edilebilir (biased/lightweight locking).

## Denemek için

```
jshell> import java.util.concurrent.locks.ReentrantLock
jshell> ReentrantLock lock = new ReentrantLock()
jshell> lock.tryLock()
jshell> lock.isHeldByCurrentThread()
jshell> lock.unlock()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html]
