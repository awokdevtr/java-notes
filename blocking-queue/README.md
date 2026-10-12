# BlockingQueue

Üretici-tüketici (`producer-consumer`) desenini elle yazmaya kalkışırsanız genelde `wait()`/`notify()` çağrıları, kilitler ve kapasite kontrolleriyle dolu, hataya açık bir kod ortaya çıkar. `java.util.concurrent.BlockingQueue`, bu senkronizasyonu kuyruğun kendi içine gömerek üreticinin ve tüketicinin birbirini beklemesini otomatik hale getirir: kuyruk doluyken ekleme, boşken alma işlemi thread'i bloke eder.

## Temel kullanım

```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

Runnable producer = () -> {
    try {
        queue.put("gorev-" + System.nanoTime());
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
};

Runnable consumer = () -> {
    try {
        String task = queue.take();
        System.out.println("Islendi: " + task);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
};
```

`put()` kuyruk doluyken, `take()` ise kuyruk boşken çağıran thread'i bekletir; her ikisi de `InterruptedException` fırlatabileceği için `interrupt` bayrağını geri ayarlamak gerekir. Kapasite sınırı (`bounded capacity`), üretimin tüketimden çok hızlı olduğu durumlarda bellek taşmasını önleyen doğal bir "geri basınç" (`backpressure`) mekanizması sağlar.

## Engellemeyen ve zaman aşımlı alternatifler

```java
boolean eklendi = queue.offer("acil-gorev", 500, TimeUnit.MILLISECONDS);
String gorev = queue.poll(1, TimeUnit.SECONDS);
```

`add()`/`remove()` kapasite dolu veya kuyruk boşken anında istisna fırlatırken, `offer()`/`poll()` belirtilen süre kadar bekleyip sonuç döner; süre dolarsa sırasıyla `false` ya da `null` verir. Bu da thread'i süresiz bloke etmeden, zaman aşımına bağlı akış kontrolü yapmayı mümkün kılar.

## Dikkat edilmesi gerekenler

- `ArrayBlockingQueue` sabit kapasiteli ve adil (`fair`) sıralama seçeneğine sahiptir; `LinkedBlockingQueue` ise varsayılan olarak sınırsızdır, kapasite belirtilmezse bellek tükenmesine kadar büyüyebilir.
- `BlockingQueue` kendi içinde thread-safe'tir; dışarıdan ek kilitleme eklemek gereksizdir ve genelde hataya yol açar.
- Üretici-tüketici sayısı birden fazla olduğunda bile kuyruk, `happens-before` ilişkisini garanti eder: bir elemanı koyan thread'in o ana kadarki yazmaları, elemanı alan thread'e görünür olur.

## Denemek için

```
jshell> import java.util.concurrent.*
jshell> var q = new ArrayBlockingQueue<Integer>(2)
jshell> q.offer(1); q.offer(2); q.offer(3)
jshell> q.poll()
```

İlk iki `offer()` çağrısı `true` döner, kuyruk dolduğu için üçüncüsü `false` döner. `poll()` ise kuyruktaki ilk elemanı (`1`) çıkarır ve bir sonraki `offer(3)` denemesi için yer açar.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingQueue.html]
