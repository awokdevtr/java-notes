# AtomicReference ile nesne referansını kilitsiz değiştirme

`AtomicInteger`/`AtomicLong` sadece sayısal bir değeri kilitsiz güncellemeyi çözer; birden fazla thread'in paylaştığı bir nesne referansını (örneğin immutable bir config veya durum nesnesi) `synchronized` yazmadan atomik olarak değiştirmek gerektiğinde `java.util.concurrent.atomic.AtomicReference<V>` devreye girer. Mantığı aynıdır: compare-and-swap (CAS), ama bu kez karşılaştırılan şey bir primitive değil referans kimliğidir.

## Kilitsiz durum değişimi

```java
record Config(int timeoutMs, boolean retryEnabled) {}

AtomicReference<Config> configRef =
    new AtomicReference<>(new Config(1000, false));

Config old = configRef.getAndSet(new Config(2000, true));
System.out.println(old.timeoutMs());           // 1000
System.out.println(configRef.get().timeoutMs()); // 2000
```

`getAndSet` eski referansı döndürüp yenisini atomik olarak yerleştirir; aradaki hiçbir thread yarı güncellenmiş bir `Config` görmez, çünkü referans değişimi tek bir CPU talimatıyla gerçekleşir. Nesnenin kendisi `immutable` (record) olduğu için bir kez okunan referans asla sonradan değişmez — bu, `AtomicReference`'ı güvenli kullanmanın ön koşuludur.

## compareAndSet ile optimistic update döngüsü

```java
AtomicReference<Config> ref = new AtomicReference<>(new Config(1000, false));

Config current;
Config updated;
do {
    current = ref.get();
    updated = new Config(current.timeoutMs() + 500, current.retryEnabled());
} while (!ref.compareAndSet(current, updated));
```

`compareAndSet(expected, newValue)` referans mevcut değer hâlâ `expected` ile aynıysa (`==`, içerik eşitliği değil) değiştirir; araya başka bir thread girip değeri değiştirmişse CAS başarısız olur ve döngü güncel değeri okuyup yeniden dener. Bu desen, `AtomicInteger.incrementAndGet()`'in iç implementasyonuyla birebir aynı retry-until-success mantığıdır, sadece sayı yerine nesne referansı üzerinde çalışır.

## Dikkat edilmesi gerekenler

- `compareAndSet` referans kimliğini (`==`) karşılaştırır; `equals()` aynı olan ama farklı nesne olan iki değer birbirinin yerine geçmez, bu yüzden döngü içinde her zaman `ref.get()` ile taze referansı okumak gerekir.
- Tutulan nesne mutable ise, `AtomicReference` sadece referans değişimini atomik yapar — nesnenin kendi alanlarına eşzamanlı yazım hâlâ veri yarışına açıktır; bu yüzden tipik kullanım immutable (record/final alanlı) nesnelerle sınırlıdır.
- İki bağımsız alanı birlikte atomik güncellemek gerekiyorsa tek bir `AtomicReference<CompositeState>` ile ikisini bir immutable nesnede birleştirmek, iki ayrı atomic alan tutmaktan daha güvenlidir.

## Denemek için

```
jshell> import java.util.concurrent.atomic.AtomicReference
jshell> var ref = new AtomicReference<>("v1")
jshell> ref.compareAndSet("v1", "v2")
jshell> ref.compareAndSet("v1", "v3")
jshell> ref.get()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html]
