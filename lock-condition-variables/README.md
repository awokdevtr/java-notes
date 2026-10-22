# Lock ve Condition ile Bekleme Kümeleri

`synchronized` bloklarında `wait()`/`notify()` kullanıldığında her monitor'ün tek bir bekleme kümesi vardır: bir üretici-tüketici kuyruğunda hem "kuyruk dolu" hem "kuyruk boş" koşulunu bekleyen thread'ler aynı havuzda birikir ve `notify()` hangisini uyandıracağını seçemez, bu da gereksiz uyanmalara ve karmaşık `if` kontrollerine yol açar. `java.util.concurrent.locks.Lock` arayüzünün `newCondition()` metodu, aynı kilit altında birden fazla bağımsız bekleme kümesi tanımlamayı sağlar.

## Birden fazla koşul, aynı kilit

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer<T> {
    private final Object[] items = new Object[100];
    private int count, putIndex, takeIndex;

    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) notFull.await();
            items[putIndex] = item;
            putIndex = (putIndex + 1) % items.length;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

`notFull` ve `notEmpty`, aynı `ReentrantLock`'a bağlı iki ayrı `Condition` nesnesidir. `put()` bir eleman eklediğinde sadece `notEmpty` üzerinde bekleyen thread'leri uyandırır; `take()` tarafında bekleyen thread'ler `notFull` havuzunda birikir ve birbirine karışmaz. `wait`/`notify` ile aynı sonucu almak için tüm thread'leri uyandırıp koşulu tekrar kontrol ettirmek gerekirdi.

## await/signal ile wait/notify arasındaki fark

`Condition.await()` çağrısı, ilişkili kilidi otomatik bırakır ve `signal()` ya da `signalAll()` ile uyandırıldığında kilidi tekrar alarak devam eder; bu, `Object.wait()`'in `synchronized` monitor üzerindeki davranışına denktir ama `Condition` ayrıca zaman aşımlı bekleme için `awaitUntil(Date)` ve kesilmeye duyarsız `awaitUninterruptibly()` gibi ek seçenekler sunar. `signal()` bekleyen thread'lerden rastgele birini uyandırırken `signalAll()` hepsini uyandırır; `notify()`/`notifyAll()` ile aynı ayrım geçerlidir, fark şu ki burada hangi bekleme kümesinin uyandırılacağı `Condition` nesnesi seçilerek netleşir. `await()` de `wait()` gibi her zaman bir döngü (`while`) içinde çağrılmalıdır, çünkü sahte uyanmalar (spurious wakeup) spesifikasyon tarafından izin verilir.

## Dikkat edilmesi gerekenler

`newCondition()` yalnızca o `Lock` nesnesi kilitliyken anlamlıdır; kilidi almadan `await()` çağırmak `IllegalMonitorStateException` fırlatır. `await()`'den dönen thread kilidi yeniden kazanmış olarak döner, ama `finally` bloğunda `unlock()` çağrılmazsa kilit sonsuza kadar tutulur. `ReentrantLock` adil (fair) modda oluşturulduysa, `Condition` üzerindeki bekleme sırası da bu adalet garantisine dahildir.

## Denemek için

```
jshell> import java.util.concurrent.locks.*
jshell> var lock = new ReentrantLock(); var cond = lock.newCondition()
jshell> lock.lock(); try { cond.awaitNanos(1); } finally { lock.unlock(); }
jshell> new Thread(() -> { lock.lock(); try { cond.signalAll(); } finally { lock.unlock(); } }).start()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/Condition.html]
