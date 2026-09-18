# CompletableFuture ile asenkron programlama temelleri

Klasik `Future.get()` çağrısı sonucu beklerken thread'i bloklar ve sonuca bağlı ek işlemleri (dönüştürme, birleştirme, hata yönetimi) zincirlemenin doğrudan bir yolunu sunmaz. `CompletableFuture`, asenkron bir görevin tamamlanmasına geri çağırma (callback) tabanlı, bloklamayan bir şekilde tepki vermeyi ve birden fazla asenkron adımı okunabilir bir zincir halinde birleştirmeyi mümkün kılar.

## supplyAsync ile görev başlatma, thenApply ile sonucu dönüştürme

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        simulateSlowLookup();
        return 42;
    })
    .thenApply(value -> "Sonuç: " + value)
    .thenApply(String::toUpperCase);

System.out.println("Görev arka planda çalışırken ana thread serbest");
System.out.println(future.get()); // "SONUÇ: 42"
```

`supplyAsync` çağrısı verilen `Supplier`'ı ortak `ForkJoinPool.commonPool()` üzerinde ayrı bir thread'de çalıştırır ve hemen bir `CompletableFuture` döndürür; ana thread bloklanmaz. `thenApply` zincirlenen her adımda önceki sonucu alıp dönüştürür ve yeni bir `CompletableFuture` üretir. Sonucu okumak için `get()` çağrısı yine de bloklar, bu yüzden gerçek bir asenkron akışta sonuç genellikle `thenAccept` veya `thenRun` gibi bir callback ile tüketilir, `get()` ile değil.

## exceptionally ve handle ile hata yönetimi

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> 10 / 0)
    .exceptionally(ex -> {
        System.out.println("Hata yakalandı: " + ex.getMessage());
        return -1;
    });

System.out.println(future.join()); // -1
```

Asenkron görev sırasında fırlatılan istisna, çağıran thread'e doğrudan ulaşmaz; bunun yerine sonuçlanan `CompletableFuture` "exceptionally completed" durumuna geçer. `exceptionally` yalnızca hata durumunda çalışan bir kurtarma değeri sağlarken, `handle` hem başarı hem hata durumunu tek bir callback'te ele almayı sağlar. `join()`, `get()`'in checked exception fırlatmayan, lambda içinde kullanımı daha kolay olan eşdeğeridir.

## Dikkat edilmesi gerekenler

- Zincirdeki `thenApply`/`thenAccept` gibi metotların "Async" olmayan versiyonları, önceki adımı tamamlayan thread üzerinde çalışır; paralellik istenmiyorsa bu davranış sürpriz olabilir.
- `thenApplyAsync` gibi Async sürümleri işi ayrı bir thread'e (varsayılan olarak `commonPool()`'a) devreder; yoğun CPU işleri için bu havuzu paylaşmak diğer asenkron görevleri de yavaşlatabilir.
- `get()` checked `ExecutionException`/`InterruptedException` fırlatır, `join()` fırlatmaz; hangisinin kullanılacağı çağrı bağlamındaki exception handling stratejisine bağlıdır.

## Denemek için

```
jshell> var f = java.util.concurrent.CompletableFuture.supplyAsync(() -> { try { Thread.sleep(1000); } catch (Exception e) {} return "hazir"; })
jshell> System.out.println("bu satir hemen basilir")
jshell> f.get()
```

İkinci satırın, birinci satırdaki görev hâlâ bir saniye boyunca arka planda beklerken anında çalıştığını gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html]
