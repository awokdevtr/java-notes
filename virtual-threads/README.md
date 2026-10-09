# virtual-threads

Geleneksel bir Java thread'i ("platform thread"), altında doğrudan bir işletim sistemi thread'ine karşılık gelir ve her biri megabaytlarca stack belleği tüketir; bu yüzden aynı anda on binlerce thread açmak pratik değildir. Oysa yüksek eşzamanlılık gerektiren sunucu kodunda (istek başına bir thread, çoğu zamanını I/O beklerken geçiren) asıl ihtiyaç ucuz, çok sayıda thread'dir. `Thread.ofVirtual()`, JVM tarafından yönetilen ve küçük sayıda platform thread üzerinde zaman paylaşımlı çalışan hafif thread'ler sunar.

## Virtual thread oluşturma

```java
Thread vt = Thread.ofVirtual().start(() -> {
    System.out.println("Çalışıyor: " + Thread.currentThread());
});
vt.join();

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofMillis(100));
            return null;
        });
    }
}
```

`Thread.ofVirtual().start(...)` doğrudan tek bir virtual thread başlatır. Gerçek kullanım genelde `Executors.newVirtualThreadPerTaskExecutor()` üzerinden olur: her görev için yeni bir virtual thread yaratılır ve havuzlama yapılmaz, çünkü virtual thread'ler zaten çok ucuzdur. Yüz bin görev gönderilse bile JVM, arka planda sadece birkaç gerçek OS thread'i ("carrier thread") kullanarak bunları sırayla taşır.

## Blocking çağrılarda "unmount" davranışı

`Thread.sleep()` veya engellenen bir I/O çağrısı gibi bloklayan bir işlem virtual thread içinde çalıştığında, JVM o virtual thread'i carrier thread'den otomatik olarak "unmount" eder; carrier thread serbest kalıp başka bir virtual thread'e hizmet eder. İşlem tamamlandığında virtual thread uygun bir carrier thread'e yeniden "mount" edilip çalışmaya devam eder. Bu sayede binlerce virtual thread, görünürde bloklayan kod yazılmasına rağmen sadece birkaç gerçek thread üzerinde verimli şekilde ilerler; geliştirici asenkron callback zinciri kurmak zorunda kalmaz.

## Dikkat edilmesi gerekenler

- `synchronized` blok içindeki bloklayan bir çağrı, virtual thread'in unmount edilmesini engeller ("pinning"); bu durumda carrier thread de bloke kalır. Yoğun bloklama olan kritik bölgelerde `synchronized` yerine `ReentrantLock` tercih edilmelidir.
- Virtual thread'ler CPU-yoğun işler için tasarlanmamıştır; asıl kazanç I/O beklemesi ağırlıklı, çok sayıda eşzamanlı görevde ortaya çıkar.
- `ThreadLocal` kullanımı virtual thread'lerde teknik olarak çalışır ama yüz binlerce thread için bellek maliyeti katlanabilir; bunun yerine `ScopedValue` gibi alternatifler düşünülebilir.

## Denemek için

```
jshell> import java.time.Duration
jshell> Thread.ofVirtual().start(() -> { Thread.sleep(Duration.ofMillis(50)); System.out.println(Thread.currentThread()); })
jshell> Thread.currentThread()
```

Çıktıdaki thread isminin `VirtualThread[...]` biçiminde olduğunu, ana thread'in ise normal bir platform thread olduğunu karşılaştırabilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html]
