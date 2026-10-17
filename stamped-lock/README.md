# stamped-lock

`ReentrantReadWriteLock`, okuyucular için gerçek paralellik sağlar ama her `readLock().lock()` çağrısı yine de bir iç sayaç güncellemesiyle maliyete yol açar; okuma çok sık, yazma nadirse bu maliyet birikir. `StampedLock`, kilit almadan okuma yapabilen bir "optimistic read" modu ekleyerek bu yükü ortadan kaldırır: okuyucu kilitlenmez, sadece okuduktan sonra arada bir yazar olup olmadığını denetler.

## Optimistic read ile kilitsiz okuma

```java
StampedLock lock = new StampedLock();
double x, y;

double distanceFromOrigin() {
    long stamp = lock.tryOptimisticRead();
    double currentX = x, currentY = y;
    if (!lock.validate(stamp)) {
        stamp = lock.readLock();
        try {
            currentX = x;
            currentY = y;
        } finally {
            lock.unlockRead(stamp);
        }
    }
    return Math.sqrt(currentX * currentX + currentY * currentY);
}
```

`tryOptimisticRead()` hiçbir kilit almaz, sadece o anki yazma sürümünü temsil eden bir `stamp` döner. Alanlar okunduktan sonra `validate(stamp)` çağrılır; bu süre içinde bir yazma kilidi alınıp bırakılmışsa `false` döner ve kod normal `readLock()`'a düşerek doğru veriyi garanti eder. Çekişmesiz senaryoda bu yol tamamen kilitsizdir ve `ReentrantReadWriteLock`'a göre çok daha az ek yük taşır.

## Yazma kilidi ve dönüştürme

```java
void move(double deltaX, double deltaY) {
    long stamp = lock.writeLock();
    try {
        x += deltaX;
        y += deltaY;
    } finally {
        lock.unlockWrite(stamp);
    }
}
```

`writeLock()` tam dışlayıcıdır ve diğer tüm okuyucu/yazıcıları bloke eder; döndürdüğü `stamp`, ilgili `unlockWrite` çağrısında kimlik doğrulaması için kullanılır. `StampedLock` ayrıca `tryConvertToWriteLock(stamp)` ile bir okuma kilidini, uygunsa kopyasız biçimde yazma kilidine yükseltmeyi destekler; bu, `ReentrantReadWriteLock`'ta deadlock'a yol açan okuma-yazma yükseltmesinin burada güvenli bir alternatifidir.

## Dikkat edilmesi gerekenler

- `StampedLock` reentrant değildir: aynı thread aynı kilidi iç içe tekrar alırsa deadlock oluşur, bu yüzden tekrar giren metotlarda `ReentrantReadWriteLock` daha güvenlidir.
- `stamp` değeri her zaman kontrol edilmelidir; `validate()` çağrısı atlanırsa optimistic read tutarsız veri döndürebilir.
- `readLock()`/`writeLock()` interrupt edilemez; kesilebilir bekleme gerekiyorsa `readLockInterruptibly()`/`writeLockInterruptibly()` kullanılmalıdır.

## Denemek için

```
jshell> import java.util.concurrent.locks.StampedLock
jshell> var lock = new StampedLock()
jshell> long s = lock.tryOptimisticRead(); System.out.println(lock.validate(s))
jshell> long w = lock.writeLock(); lock.unlockWrite(w); System.out.println(lock.validate(s))
```

İlk `validate` `true` döner çünkü arada yazma olmadı; yazma kilidi alınıp bırakıldıktan sonra aynı `stamp` ile yapılan ikinci `validate` artık `false` döner.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/StampedLock.html]
