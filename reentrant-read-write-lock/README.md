# reentrant-read-write-lock

`ReentrantLock` ve `synchronized`, okuma ve yazma işlemlerini aynı tek kilitle korur: veriyi sadece okuyan iki thread birbirini bloke etmemesi gerekirken, tek kilit onları da sıraya koyar. `ReentrantReadWriteLock`, okuma ve yazma erişimini ayrı kilitlere bölerek okuma ağırlıklı senaryolarda paralelliği artırır.

## Temel kullanım

```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
Map<String, Integer> cache = new HashMap<>();

Integer get(String key) {
    readLock.lock();
    try {
        return cache.get(key);
    } finally {
        readLock.unlock();
    }
}

void put(String key, Integer value) {
    writeLock.lock();
    try {
        cache.put(key, value);
    } finally {
        writeLock.unlock();
    }
}
```

`readLock()` birden fazla thread tarafından aynı anda tutulabilir; hiçbir yazar aktif olmadığı sürece okuyucular birbirini bloke etmez. `writeLock()` ise tam dışlayıcıdır: bir thread yazma kilidini tutarken hem diğer yazarlar hem de okuyucular bekler. Her `lock()` çağrısının karşılığı `finally` içinde `unlock()` ile kapatılmalıdır, aksi halde kilit sonsuza kadar tutulu kalır.

## Yazardan okumaya kilit düşürme

```java
writeLock.lock();
try {
    cache.put("key", 1);
    readLock.lock();
} finally {
    writeLock.unlock();
}
try {
    // artik sadece okuma kilidi tutuluyor, baska yazar bekliyor
    process(cache.get("key"));
} finally {
    readLock.unlock();
}
```

Bu kalıba `lock downgrading` denir: yazma kilidi tutulurken okuma kilidi alınır, ardından yazma kilidi serbest bırakılır. Böylece veriyi tazeleyip hemen ardından güvenle okunabilir; aradaki anda başka bir yazarın araya girip veriyi değiştirmesi engellenir. Tersi yönde yükseltme (okuma kilidinden yazma kilidine geçiş) desteklenmez ve deadlock'a yol açar.

## Dikkat edilmesi gerekenler

- Bir thread okuma kilidini tutarken yazma kilidi almaya çalışırsa (lock upgrade), tek thread olduğu durumlarda kendi kendini bloke eder çünkü yazma kilidi aktif okuyucuları bekler.
- Varsayılan mod adil değildir (`non-fair`); sürekli yazma isteği gelen bir sistemde okuyucular açlık (`starvation`) çekebilir. Adil mod için `new ReentrantReadWriteLock(true)` kullanılır.
- Her iki kilit de reentrant'tır: aynı thread aynı kilidi iç içe tekrar alabilir, ancak her `lock()` çağrısının bir `unlock()` ile eşleşmesi gerekir.

## Denemek için

```
jshell> import java.util.concurrent.locks.ReentrantReadWriteLock
jshell> var rw = new ReentrantReadWriteLock()
jshell> rw.readLock().lock(); System.out.println("okuma kilidi sayisi: " + rw.getReadLockCount())
jshell> rw.readLock().unlock()
```

`getReadLockCount()` ile aynı anda kaç thread'in okuma kilidini tuttuğunu gözlemleyebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReentrantReadWriteLock.html]
