# ExecutorService ve Thread Pool Temelleri

Her görev için `new Thread(...).start()` çağırmak basit görünür ama gerçek dünyada maliyetlidir: her `Thread` bir işletim sistemi kaynağıdır, yaratılması ve bağlam değişimi (`context switch`) pahalıdır; görev sayısı arttığında kontrolsüzce thread açmak JVM'i tüketir. `ExecutorService`, thread yaşam döngüsünü programcıdan alıp sabit sayıda worker thread'i yeniden kullanan bir havuz (`pool`) üzerinden görev çalıştırmayı sağlar.

## Thread yerine görev gönderme

```java
ExecutorService havuz = Executors.newFixedThreadPool(4);

Future<Integer> sonuc = havuz.submit(() -> {
    Thread.sleep(100);
    return 42;
});

System.out.println(sonuc.get());
```

`submit()` bir `Runnable` veya `Callable` kabul eder ve hemen bir `Future` döndürür; çağıran thread bloklanmaz, görev havuzdaki uygun bir worker thread'e atanana kadar beklemede kalır. `Callable.call()` bir değer döndürebildiği ve `checked exception` fırlatabildiği için `Runnable.run()`'dan daha esnektir. `Future.get()` çağrısı, sonuç hazır olana kadar bloklar ve görev sırasında bir istisna fırlatıldıysa onu `ExecutionException` içine sararak yeniden fırlatır.

## Kapatma: shutdown vs shutdownNow

```java
havuz.shutdown();
if (!havuz.awaitTermination(5, TimeUnit.SECONDS)) {
    havuz.shutdownNow();
}
```

`shutdown()`, kuyruktaki mevcut görevlerin tamamlanmasına izin verir ama yeni görev kabul etmeyi durdurur; `shutdownNow()` ise çalışan thread'leri `interrupt()` ederek görevleri yarıda kesmeye çalışır ve bekleyen görevleri bir listeye koyup döndürür. `awaitTermination`, belirtilen sürede kapanma tamamlanırsa `true` döner; aksi halde havuz hâlâ görev işliyor demektir.

## Dikkat edilmesi gerekenler

- `ExecutorService`'in worker thread'leri varsayılan olarak `daemon` değildir; `shutdown()` çağrılmazsa JVM, `main` metodu bitse bile havuz canlı olduğu için sonlanmaz.
- `Executors.newCachedThreadPool()` sınırsız sayıda thread yaratabilir; patlayan görev sayısı doğrudan `OutOfMemoryError`'a yol açabilir.
- `Executors.newFixedThreadPool()` arkasında sınırsız (`unbounded`) bir `LinkedBlockingQueue` kullanır; worker'lar görevleri yeterince hızlı işleyemezse kuyruk sürekli büyür ve bellek tükenebilir. Üretimde genellikle sınırlı kuyruklu `ThreadPoolExecutor` doğrudan tercih edilir.

## Denemek için

```
jshell> var havuz = java.util.concurrent.Executors.newFixedThreadPool(2)
jshell> var f = havuz.submit(() -> { Thread.sleep(200); return "tamam"; })
jshell> f.get()
jshell> havuz.shutdown()
```

`f.get()` çağrısının yaklaşık 200 milisaniye bekleyip `"tamam"` döndürdüğünü, bu sırada ana thread'in bloklandığını ama havuzdaki diğer worker thread'in boşta kaldığını gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html]
