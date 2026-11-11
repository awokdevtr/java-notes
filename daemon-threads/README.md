# daemon-threads

Bir JVM süreci, çalışan en az bir `user thread` olduğu sürece hayatta kalır; `main` metodu bitse bile arka planda başlattığınız bir thread çalışıyorsa program kapanmaz. Bu, loglama, önbellek temizleme ya da periyodik ping gönderme gibi "yardımcı" işler için istenmeyen bir davranıştır: uygulamanın asıl işi bittiğinde bu yardımcı thread'lerin süreci canlı tutmasını istemezsiniz. `daemon thread`, tam olarak bu sorunu çözer.

## Temel kullanım

```java
Thread worker = new Thread(() -> {
    while (true) {
        System.out.println("arka planda çalışıyorum...");
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            return;
        }
    }
});
worker.setDaemon(true);
worker.start();

System.out.println("main metodu bitti");
```

`setDaemon(true)` çağrısı, thread'i `daemon thread` olarak işaretler. JVM, kalan tüm thread'lerin daemon olup olmadığını kontrol eder; eğer ortada hiç `user thread` kalmamışsa, daemon thread'ler `finally` blokları çalıştırılmadan, hatta sonsuz döngünün ortasında olsalar bile aniden durdurularak JVM kapatılır. Yukarıdaki örnekte `main` bittiği anda `worker`'ın döngüsü tamamlanmamış olsa da süreç sonlanır.

## Devralma ve zamanlama kısıtı

```java
Thread parent = new Thread(() -> {
    Thread child = new Thread(() -> {});
    System.out.println(child.isDaemon()); // parent daemon ise true
});
parent.setDaemon(true);
parent.start();
```

Yeni oluşturulan bir thread, varsayılan olarak onu yaratan thread'in `daemon` durumunu devralır. `setDaemon()` ise sadece thread henüz `start()` edilmeden önce çağrılabilir; thread başladıktan sonra bu metodu çağırmak `IllegalThreadStateException` fırlatır, çünkü JVM thread'in daemon durumunu çalışma zamanında değiştirmeye güvenli bir şekilde izin vermez.

## Ne zaman işe yarar

- Periyodik bakım işleri (önbellek süpürme, metrik gönderme, bağlantı havuzu temizliği) için idealdir: uygulama kapanırken bu işlerin tamamlanmasını beklemeniz gerekmez.
- Kritik iş mantığı (dosyaya yazma, veritabanı commit'i) asla daemon thread üzerinde çalıştırılmamalıdır; JVM onu yarıda kesebilir ve veri kaybına yol açabilir.
- `ExecutorService` kullanırken thread havuzundaki thread'lerin daemon olup olmaması, havuzu oluşturan `ThreadFactory`'ye bağlıdır; varsayılan `Executors` fabrikaları `user thread` üretir, bu yüzden havuzu `shutdown()` etmeyi unutmak JVM'in kapanmasını engelleyebilir.

## Denemek için

```
jshell> Thread t = new Thread(() -> { while (true) {} }); t.setDaemon(true); t.start(); System.out.println(t.isDaemon())
```

`true` çıktısını görürsünüz. Aynı kodu `setDaemon(true)` satırı olmadan ayrı bir `.java` dosyasında çalıştırırsanız, sonsuz döngü `user thread` olarak kaldığı için JVM hiçbir zaman kapanmaz; `jshell` içinde bu fark arka plan oturumu nedeniyle doğrudan gözlemlenemez ama derleyip `java` ile çalıştırdığınızda süreci `Ctrl+C` ile kesmeniz gerektiğini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html#setDaemon(boolean)]
