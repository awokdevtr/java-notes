# Runnable vs Callable

Bir görevi başka bir thread'de veya bir `ExecutorService` üzerinde çalıştırmak istediğinizde Java iki arayüz sunar: sonuç döndürmeyen ve istisna fırlatamayan `Runnable`, ya da sonuç döndürebilen ve `checked exception` fırlatabilen `Callable`. Yanlış arayüzü seçmek, bir hesaplamanın sonucunu elde etmek için paylaşımlı değişkenlere ve el yapımı senkronizasyona başvurmaya ya da istisnaları yutmaya zorlar.

## Runnable: sonuçsuz, istisnasız görev

```java
Runnable gorev = () -> {
    System.out.println("arka planda calisiyor");
};

Thread t = new Thread(gorev);
t.start();
```

`Runnable` arayüzü tek bir `void run()` metodu tanımlar; parametre almaz, değer döndürmez ve imzasında hiçbir `checked exception` bildirmez. Bu yüzden `run()` içinde bir `IOException` fırlatılırsa ya `try-catch` ile yakalanıp yutulmalı ya da `unchecked` bir istisnaya (`RuntimeException`) çevrilmelidir. `Thread` constructor'ı ve `ExecutorService.execute()` metodu `Runnable` bekler çünkü bu çağrı yollarının geri dönecek bir sonucu yoktur.

## Callable: sonuç ve checked exception

```java
Callable<Integer> gorev = () -> {
    Thread.sleep(100);
    return 42;
};

ExecutorService havuz = Executors.newSingleThreadExecutor();
Future<Integer> sonuc = havuz.submit(gorev);
System.out.println(sonuc.get());
havuz.shutdown();
```

`Callable<V>` arayüzü `V call() throws Exception` imzasına sahiptir; bu yüzden hem bir değer döndürebilir hem de `checked exception`'ları doğrudan (sarmadan) fırlatabilir. `ExecutorService.submit(Callable<V>)` bu görevi havuza gönderir ve hemen bir `Future<V>` döndürür; gerçek sonuç `get()` çağrıldığında, görev bitmemişse çağıran thread'i bloke ederek elde edilir. Görev sırasında bir istisna fırlarsa `get()` bunu `ExecutionException` içine sararak yeniden fırlatır.

## Dikkat edilmesi gerekenler

- `submit(Runnable)` de bir `Future<?>` döndürür ama bu `Future`'ın `get()`'i her zaman `null` verir; gerçek bir sonuç istiyorsanız `Callable` kullanmalısınız.
- `Callable`'ı `Thread` constructor'ına doğrudan veremezsiniz; `Thread` sadece `Runnable` kabul eder. İkisi arasında köprü kurmak için `Executors.callable(Runnable, result)` kullanılabilir.
- `get()`'in fırlattığı `ExecutionException`'ın asıl nedeni `getCause()` ile alınır; orijinal istisnayı kaybetmemek için bu adımı atlamayın.

## Denemek için

```
jshell> import java.util.concurrent.*
jshell> var havuz = Executors.newSingleThreadExecutor()
jshell> Future<Integer> f = havuz.submit(() -> 21 + 21)
jshell> f.get()
```

`submit` çağrısı hemen döner, `f.get()` ise sonucu hazır olana kadar bekler ve `42` değerini verir — `Runnable` ile bu değeri doğrudan almanın bir yolu yoktur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Callable.html]
