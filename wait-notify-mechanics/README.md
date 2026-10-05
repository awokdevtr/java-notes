# wait, notify ve notifyAll

Bir thread, başka bir thread'in bir koşulu değiştirmesini beklerken CPU'yu `while (!hazır) {}` gibi bir döngüyle sürekli yoklamak (busy-waiting) işlemciyi gereksiz yere tüketir. `Object` sınıfının `wait()`, `notify()` ve `notifyAll()` metotları, bir thread'i monitor üzerinde gerçekten uykuya yatırıp başka bir thread koşulu değiştirdiğinde onu uyandırmayı sağlayan düşük seviye koordinasyon mekanizmasıdır.

## Temel kullanım: üretici-tüketici

```java
class Depo {
    private final Queue<Integer> eleman = new LinkedList<>();
    private final int kapasite = 5;

    synchronized void koy(int deger) throws InterruptedException {
        while (eleman.size() == kapasite) {
            wait();
        }
        eleman.add(deger);
        notifyAll();
    }

    synchronized int al() throws InterruptedException {
        while (eleman.isEmpty()) {
            wait();
        }
        int deger = eleman.poll();
        notifyAll();
        return deger;
    }
}
```

`wait()` çağrıldığında thread, çağrıldığı nesnenin monitor kilidini bırakır ve o nesnenin bekleme kümesine (wait set) girer; `notify()`/`notifyAll()` ile uyandırılana kadar bloke kalır. Her ikisi de ancak çağıran thread o nesnenin kilidine zaten sahipken geçerlidir, aksi halde `IllegalMonitorStateException` fırlatılır. Uyanan thread kilidi otomatik olarak geri kazanır ve koşulu `while` ile tekrar kontrol eder; bu kontrol atlanırsa "spurious wakeup" (sebepsiz uyanma) veya başka bir thread'in koşulu tekrar bozması durumunda hatalı ilerleme olur.

## Dikkat edilmesi gerekenler

- `notify()` bekleyen thread'lerden rastgele birini uyandırır, `notifyAll()` ise hepsini; yanlış thread'in uyanıp koşulu uygun bulamadan tekrar uyuması riskini azaltmak için genellikle `notifyAll()` tercih edilir.
- Koşul kontrolü her zaman `if` değil `while` ile yapılmalıdır; `wait()`'in beklenmedik şekilde dönmesi JLS tarafından açıkça izin verilen bir davranıştır.
- `notify()` çağrısı, bekleyen hiçbir thread yokken yapılırsa kaybolur ("lost notify"); bu yüzden bildirim göndermeden önce durumu değiştirmek ve kilidi elden bırakmadan `notify` yapmak önemlidir.
- Günümüzde doğrudan `wait`/`notify` yazmak yerine `java.util.concurrent` paketindeki `BlockingQueue`, `CountDownLatch` veya `Condition` gibi daha yüksek seviyeli, hataya daha az açık araçlar tercih edilir.

## Denemek için

```
jshell> Object kilit = new Object()
jshell> Thread bekleyen = new Thread(() -> { synchronized (kilit) { try { System.out.println("bekliyor"); kilit.wait(); System.out.println("uyandı"); } catch (InterruptedException e) {} } })
jshell> bekleyen.start(); Thread.sleep(500)
jshell> synchronized (kilit) { kilit.notify(); }
```

`bekleyen` thread'i başlatıldığında "bekliyor" yazdırıp bloke olur; ana thread `notify()` çağırana kadar "uyandı" satırı hiç basılmaz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#wait()]
