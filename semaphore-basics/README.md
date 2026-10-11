# semaphore-basics

Birden fazla thread'in paylaşılan, sınırlı sayıda kaynağa (bağlantı havuzu, sınırlı sayıda işçi slotu gibi) aynı anda erişmesini kısıtlamak gerektiğinde `synchronized` yalnızca tek bir thread'e izin verdiği için yetersiz kalır. `Semaphore`, belirli sayıda "izin" (permit) tutan ve bu izinleri thread'ler arasında dağıtan bir senkronizasyon aracıdır.

## Temel kullanım

```java
Semaphore semaphore = new Semaphore(2);

Runnable worker = () -> {
    try {
        semaphore.acquire();
        System.out.println(Thread.currentThread().getName() + " kaynagi kullaniyor");
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        semaphore.release();
    }
};

for (int i = 0; i < 5; i++) {
    new Thread(worker).start();
}
```

`new Semaphore(n)` ile n izin oluşturulur. `acquire()` bir izin alana kadar bloke olur; izin yoksa thread bekler. `release()` izni geri verir ve bekleyen bir thread'i uyandırır. Aynı anda en fazla n thread kritik bölgeye girebilir, bu yüzden `release()` her zaman `finally` içinde çağrılmalıdır.

## Zaman aşımlı ve adil erişim

```java
Semaphore fairSemaphore = new Semaphore(1, true);

if (fairSemaphore.tryAcquire(500, TimeUnit.MILLISECONDS)) {
    try {
        // kritik bolge
    } finally {
        fairSemaphore.release();
    }
} else {
    System.out.println("Izin alinamadi, zaman asimi");
}
```

İkinci constructor parametresi `true` verildiğinde adil (`fair`) mod aktif olur ve izinler FIFO sırasına göre dağıtılır; aksi halde JVM thread'leri herhangi bir sırayla ödüllendirebilir. `tryAcquire(long, TimeUnit)` belirtilen sürede izin alınamazsa thread'i süresiz bloke etmeden `false` döner.

## Dikkat edilmesi gerekenler

- `Semaphore` kilidi hangi thread'in aldığını takip etmez; bir thread izin alıp başka bir thread o izni serbest bırakabilir. Bu, `ReentrantLock`'tan farklı olarak sahiplik kontrolü sağlamaz.
- Başlangıç izin sayısı 1 olduğunda `Semaphore` karşılıklı dışlama (`mutual exclusion`) sağlar, ancak yeniden giriş (`reentrant`) desteklemez: aynı thread ikinci kez `acquire()` çağırırsa kendi kendini bloke edebilir.
- `release()` çağrı sayısı `acquire()` sayısından fazla olursa izin sayısı başlangıç değerinin üzerine çıkabilir; bu genelde bir programlama hatasının işaretidir.

## Denemek için

```
jshell> import java.util.concurrent.Semaphore
jshell> var sem = new Semaphore(1)
jshell> sem.acquire(); System.out.println("izin alindi, kalan: " + sem.availablePermits())
jshell> sem.release(); System.out.println("izin birakildi, kalan: " + sem.availablePermits())
```

`availablePermits()` ile kalan izin sayısını her adımda gözlemleyebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Semaphore.html]
