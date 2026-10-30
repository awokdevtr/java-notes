# scheduled-executor-service

Belirli bir süre sonra veya periyodik olarak kod çalıştırmak için `Thread.sleep` ile kendi döngünüzü yazmak hem kaba hem de hataya açıktır: sapma birikir, istisnalar thread'i sessizce öldürür, iptal mekanizması yoktur. `java.util.Timer` da tek thread üzerinde çalıştığı ve bir görevdeki yakalanmamış istisnanın tüm zamanlayıcıyı durdurduğu için kırılgandır. `ScheduledExecutorService`, zamanlamayı bir thread havuzuna devrederek bu sorunları çözer.

## Temel kullanım

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

scheduler.schedule(() -> System.out.println("3 saniye sonra calisti"),
        3, TimeUnit.SECONDS);

ScheduledFuture<?> periodic = scheduler.scheduleAtFixedRate(
        () -> System.out.println("heartbeat"),
        0, 1, TimeUnit.SECONDS);
```

`schedule()` görevi bir defa, belirtilen gecikmeden sonra çalıştırır ve bir `ScheduledFuture` döner; bu nesne üzerinden sonucu bekleyebilir ya da `cancel()` ile görevi iptal edebilirsiniz. Havuzdaki thread sayısı sabittir, dolayısıyla aynı anda çalışabilecek görev sayısı da bu sayıyla sınırlıdır.

## scheduleAtFixedRate vs scheduleWithFixedDelay

```java
scheduler.scheduleAtFixedRate(task, 0, 1, TimeUnit.SECONDS);
scheduler.scheduleWithFixedDelay(task, 0, 1, TimeUnit.SECONDS);
```

`scheduleAtFixedRate`, bir sonraki çalışmayı önceki çalışmanın *başlangıç* zamanına göre planlar; görev periyottan uzun sürerse yeni çağrı bir öncekinin bitmesini bekler ve art arda, aradaki gecikme olmadan tetiklenir (eş zamanlı çalışmaz, ama birikir). `scheduleWithFixedDelay` ise bir sonraki çalışmayı önceki görevin *bitiş* zamanından itibaren sayar, bu yüzden görev süresi periyodu her zaman etkiler. Görev süresi öngörülemezse `scheduleWithFixedDelay` birikmeyi önler.

## Dikkat edilmesi gerekenler

- Periyodik bir görev çalışırken yakalanmamış bir `RuntimeException` fırlatırsa, o görevin gelecekteki tüm tekrarları sessizce iptal olur; hiçbir log veya istisna görünmez. Görev gövdesini kendi `try/catch`'inizle sarmak zorunludur.
- `ScheduledExecutorService` thread'leri daemon değildir; `shutdown()` çağrılmadan JVM kapanmaz.
- Tek thread'lik bir havuzda uzun süren bir görev, aynı havuzdaki diğer tüm zamanlanmış görevleri geciktirir.

## Denemek için

```
jshell> import java.util.concurrent.*
jshell> var s = Executors.newScheduledThreadPool(1)
jshell> s.scheduleAtFixedRate(() -> System.out.println("tick " + System.currentTimeMillis()), 0, 500, TimeUnit.MILLISECONDS)
```

Yarım saniyede bir "tick" satırının yazdığını, `s.shutdown()` çağırana kadar devam ettiğini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ScheduledExecutorService.html]
