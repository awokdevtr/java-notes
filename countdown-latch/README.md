# countdown-latch

Birden fazla thread'in, başka thread'lerin belirli bir sayıda işi tamamlamasını beklemesi gerektiğinde elle yazılmış `wait`/`notifyAll` kodu hem kırılgandır hem de kaybolan bildirim gibi tuzaklara açıktır. `CountDownLatch`, bir sayacı sıfıra inene kadar bekleyen thread'leri bloke eden, tek kullanımlık ve basit bir senkronizasyon aracı sunar.

## Temel kullanım

```java
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        System.out.println("Worker tamamlandi");
        latch.countDown();
    }).start();
}

latch.await();
System.out.println("Tum worker'lar bitti, ana thread devam ediyor");
```

Sayaç `new CountDownLatch(n)` ile başlangıç değerine ayarlanır. Her `countDown()` çağrısı sayacı bir azaltır; sayaç sıfıra ulaştığında `await()` içinde bloke olan tüm thread'ler aynı anda serbest kalır. `await()` birden fazla thread tarafından çağrılabilir, hepsi aynı olayı bekler.

## Zaman aşımlı bekleme

```java
boolean completed = latch.await(2, TimeUnit.SECONDS);
if (!completed) {
    System.out.println("Zaman asimi: bazi worker'lar hala calisiyor");
}
```

`await(long, TimeUnit)` aşırı yüklemesi, sayaç sıfıra inmeden belirtilen süre geçerse `false` döndürür ve thread'i süresiz bloke etmez. Bu, bağımlı servislerin sonsuza kadar beklemesini önlemek için tercih edilir.

## Dikkat edilmesi gerekenler

- `CountDownLatch` tek kullanımlıktır: sayaç sıfıra indikten sonra sıfırlanamaz. Tekrarlanan senkronizasyon gerekiyorsa `CyclicBarrier` kullanılmalıdır.
- `countDown()` sayaç zaten sıfırdayken çağrılırsa hiçbir etkisi olmaz, istisna fırlatmaz.
- Başlangıç sayısı kadar `countDown()` çağrısı garanti edilmelidir; aksi halde `await()` eden thread'ler kalıcı olarak bloke kalabilir, bu yüzden `countDown()` genelde `finally` bloğunda çağrılır.

## Denemek için

```
jshell> import java.util.concurrent.CountDownLatch
jshell> var latch = new CountDownLatch(2)
jshell> new Thread(() -> { System.out.println("isci-1"); latch.countDown(); }).start()
jshell> latch.await(); System.out.println("tek countDown sonrasi hala " + latch.getCount())
```

İkinci `countDown()` çağrılmadan `await()`'in bloke olmaya devam ettiğini, `getCount()` ile kalan sayacı gözlemleyebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CountDownLatch.html]
