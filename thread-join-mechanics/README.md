# thread-join-mechanics

Bir thread başka bir thread'in işini bitirmesini beklemek istediğinde en doğal çözüm `join()` çağrısıdır. `Thread.sleep()` ile sabit bir süre beklemek hem kırılgandır (iş erken biterse zaman kaybedilir, gecikirse sonuç hazır olmadan okunur) hem de doğru değildir; `join()` ise çağrılan thread'in gerçekten `TERMINATED` durumuna geçmesini garanti eder.

## Temel kullanım

```java
Thread worker = new Thread(() -> {
    try { Thread.sleep(500); } catch (InterruptedException ignored) {}
    System.out.println("is tamamlandi");
});
worker.start();
worker.join();
System.out.println("ana thread devam ediyor");
```

`join()` çağrısı, çağıran thread'i (burada ana thread) `worker` sonlanana kadar bloklar. `worker.join()` dönmeden "ana thread devam ediyor" satırı asla basılmaz; bu, `join()`'in bellek görünürlüğü açısından da bir garanti taşıdığı anlamına gelir: `worker` içinde yazılan alanlar, `join()`'den sonra çağıran thread'e `happens-before` ilişkisiyle güvenle görünür olur, ek bir `volatile` ya da kilide gerek kalmaz.

## Zaman aşımlı join ve kesme

```java
Thread worker = new Thread(() -> { /* uzun is */ });
worker.start();
worker.join(2000);
if (worker.isAlive()) {
    System.out.println("2 saniyede bitmedi, devam ediyor");
}
```

`join(millis)` overload'u, belirtilen süre geçtiğinde thread hâlâ çalışıyor olsa bile bloklamayı sonlandırır; bu yüzden dönüşten sonra işin bittiğini varsaymadan `isAlive()` ile kontrol etmek gerekir. `join()` bekleme sırasında kesilirse (`interrupt()`), bloklanmış thread `InterruptedException` fırlatır ve kesme bayrağı temizlenir; bu da `join()`'i, `wait()` gibi kesilebilir blok noktalarından biri yapar.

## Dikkat edilmesi gerekenler

- `join()`, çağrılan thread'i durdurmaz; sadece çağıran thread'i bekletir. Thread'i durdurmak için paylaşımlı bir bayrak veya `interrupt()` kullanılmalıdır.
- Birden fazla thread'i sırayla `join()` etmek, aralarında bağımlılık yoksa toplam bekleme süresini gereksiz yere uzatır; bağımsız işlerde önce hepsini `start()` edip sonra hepsini `join()` etmek daha doğrudur.
- Ana thread kendi kendini `join()` etmeye çalışırsa (`Thread.currentThread().join()`), sonsuza kadar bloklanır; bu bir deadlock'tur ve derleyici bunu engellemez.

## Denemek için

```
jshell> Thread t = new Thread(() -> { try { Thread.sleep(1000); } catch (InterruptedException e) {} System.out.println("bitti"); })
jshell> t.start(); t.join(); System.out.println("sonra")
```

"bitti" satırının "sonra"dan önce basıldığını görürsünüz. `t.join()` satırını silip tekrar denerseniz, `t.start()` sonrası `System.out.println("sonra")` hemen çalışır ve "bitti" ondan sonra rastgele bir anda görünür.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html#join()]
