# exchanger-thread-handoff

İki thread'in birbirine veri aktarması genelde bir `BlockingQueue` üzerinden yapılır, ama bu tek yönlü akış için tasarlanmıştır: üretici koyar, tüketici alır. İki thread'in aynı anda birbirine karşılıklı veri vermesi gerektiğinde (örneğin bir arabellek takası ya da ping-pong tarzı bir protokol) kuyruk fazla kalır. `Exchanger`, tam da bu simetrik takas senaryosu için var: iki thread `exchange()`'i çağırır, her ikisi de gelene kadar bloke olur, sonra karşı tarafın verdiği nesneyle geri döner.

## İki thread arasında nesne takası

```java
Exchanger<StringBuilder> exchanger = new Exchanger<>();

Runnable producer = () -> {
    StringBuilder buffer = new StringBuilder("yeni veri");
    try {
        StringBuilder emptyBuffer = exchanger.exchange(buffer);
        emptyBuffer.setLength(0); // boşaltılan arabelleği yeniden kullan
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
};

Runnable consumer = () -> {
    StringBuilder emptyBuffer = new StringBuilder();
    try {
        StringBuilder filled = exchanger.exchange(emptyBuffer);
        System.out.println(filled); // "yeni veri"
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
};
```

`exchange(buffer)` çağrısı, karşı taraf da kendi `exchange()` çağrısını yapana kadar bloke olur; iki thread eşleştiği anda her biri kendi verdiği nesne yerine karşı tarafınkini alır. Bu, çift tarafli bir randevu noktasıdır: `CyclicBarrier` gibi N thread'i senkronize etmez, sadece tam iki thread'i eşleştirir ve aralarında veri taşır.

## Zaman aşımlı takas

```java
try {
    StringBuilder result = exchanger.exchange(new StringBuilder(), 500, TimeUnit.MILLISECONDS);
} catch (TimeoutException e) {
    // 500ms içinde karşı taraf gelmediyse buraya düşer
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

Süresiz `exchange()` karşı taraf hiç gelmezse kalıcı olarak bloke eder; zaman aşımlı aşırı yüklenmiş sürüm, bir `TimeoutException` fırlatarak thread'in sonsuza kadar beklemesini önler. Bu, üretici/tüketici taraflarından biri çöktüğünde veya yavaş kaldığında deadlock riskini ortadan kaldırır.

## Ne zaman işe yarar

- İki thread arasında tekrar kullanılabilir arabellek (buffer) takası yaparak nesne oluşturma/çöp toplama yükünü azaltmak.
- Genetik algoritmalarda iki "worker" thread'in ara sonuçlarını karşılıklı değiştirmesi gibi simetrik pipeline tasarımlarında.
- Üçten fazla thread senkronize etmek gerekiyorsa `Exchanger` uygun değildir; bu durumda `CyclicBarrier` veya `Phaser` kullanılmalıdır.

## Denemek için

```
jshell> import java.util.concurrent.Exchanger
jshell> var ex = new Exchanger<String>()
jshell> new Thread(() -> { try { System.out.println("A aldı: " + ex.exchange("A'dan")); } catch (Exception e) {} }).start()
jshell> System.out.println("B aldı: " + ex.exchange("B'den"))
```

İlk thread `exchange("A'dan")` ile bloke olur, ana thread `exchange("B'den")` çağırdığı anda ikisi eşleşir: A konsolda "B'den" yazdığını, ana thread ise "A'dan" aldığını görür.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Exchanger.html]
