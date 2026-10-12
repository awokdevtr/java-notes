# cyclic-barrier

`CountDownLatch` tek kullanımlıktır: sayaç sıfıra indikten sonra bir daha sıfırlanamaz. Birden fazla thread'in her turda birbirini beklediği, tekrarlı aşamalı işler için (örneğin bir simülasyonun her adımında tüm worker'ların bitmesini bekleyip bir sonraki adıma geçmek) bu yetersizdir. `CyclicBarrier`, belirli sayıda thread belirlenen noktaya ulaştığında hepsini aynı anda serbest bırakan ve ardından otomatik olarak kendini sıfırlayan, yeniden kullanılabilir bir senkronizasyon noktası sunar.

## Temel kullanım

```java
CyclicBarrier barrier = new CyclicBarrier(3, () ->
    System.out.println("Tum worker'lar bu turu tamamladi, siradaki tura geciliyor"));

for (int i = 0; i < 3; i++) {
    int id = i;
    new Thread(() -> {
        for (int round = 0; round < 2; round++) {
            System.out.println("Worker " + id + " tur " + round + " calisiyor");
            try {
                barrier.await();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }).start();
}
```

`new CyclicBarrier(n, action)` çağrısında `n`, bariyerde buluşması gereken thread sayısıdır; isteğe bağlı `action`, son thread bariyere ulaştığında yalnızca bir kez, o thread tarafından çalıştırılır. Her thread kendi işini bitirince `await()` çağırır ve son thread gelene kadar bloke olur; `n`. thread geldiğinde hepsi serbest kalır ve bariyer otomatik olarak sıfırlanır, böylece aynı `barrier` nesnesi bir sonraki tur için tekrar kullanılabilir.

## Kırık bariyer durumu

```java
try {
    barrier.await(2, TimeUnit.SECONDS);
} catch (BrokenBarrierException e) {
    System.out.println("Baska bir thread bariyeri kirdi, bu tur gecersiz");
} catch (TimeoutException e) {
    System.out.println("Zaman asimi: bazi worker'lar hala gelmedi");
}
```

Bekleyen thread'lerden biri kesilirse (`InterruptedException`) veya zaman aşımına uğrarsa, bariyerdeki diğer tüm thread'ler için bariyer `broken` duruma geçer ve `BrokenBarrierException` fırlatılır. Bu, tek bir thread'in hatasının sessizce diğerlerini sonsuza kadar bloke etmesini önler; kırık bir bariyer yalnızca `reset()` ile elle düzeltilebilir.

## Dikkat edilmesi gerekenler

- `CyclicBarrier(n)`, tam olarak `n` thread'in `await()` çağırmasını bekler; `n`'den az thread varsa diğerleri süresiz bloke kalır.
- Bariyer eylemi (`action`), son thread'in kendi thread'inde çalışır; uzun sürmesi o thread'i geciktirir.
- `CountDownLatch` tek olaylık, sıfırlanamaz bekleme için uygundur; thread sayısı sabitken tekrarlanan senkronizasyon noktaları gerektiğinde `CyclicBarrier` tercih edilmelidir.

## Denemek için

```
jshell> import java.util.concurrent.CyclicBarrier
jshell> var barrier = new CyclicBarrier(2, () -> System.out.println("herkes geldi"))
jshell> new Thread(() -> { try { barrier.await(); } catch (Exception e) {} }).start()
jshell> barrier.await()
```

İlk `await()` diğer thread'i beklerken bloke olur; ikinci thread `await()` çağırdığında her ikisi de serbest kalır ve `action` tetiklenir. Ardından `barrier.getParties()` ile toplam thread sayısını, `barrier.await()`'i tekrar çağırarak bariyerin yeniden kullanılabildiğini gözlemleyebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CyclicBarrier.html]
