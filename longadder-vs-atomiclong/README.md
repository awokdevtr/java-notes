# LongAdder ve AtomicLong

`AtomicLong`, bir sayaç üzerinde `compareAndSet` ile kilitsiz güncelleme sağlar; ama yüzlerce thread aynı tek `long` alanına aynı anda yazmaya çalıştığında CAS sürekli çakışır, başarısız olan her thread güncel değeri okuyup yeniden dener. Yüksek çekişme (contention) altında bu retry döngüsü, beklenenin tersine, kilitten daha fazla CPU harcayabilir. `java.util.concurrent.atomic.LongAdder`, tek bir değişken yerine thread'lere dağıtılmış birden fazla hücre (`Cell`) tutarak bu çekişmeyi dağıtır.

## Tek değişken yerine dağıtılmış hücreler

```java
LongAdder hits = new LongAdder();

Runnable worker = () -> {
    for (int i = 0; i < 100_000; i++) {
        hits.increment();
    }
};

Thread t1 = new Thread(worker);
Thread t2 = new Thread(worker);
t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(hits.sum()); // 200000
```

`increment()` çağrıldığında `LongAdder`, thread'e özgü bir `Cell`'i günceller; çekişme algılandığında dahili hücre dizisini büyütüp thread'leri farklı hücrelere yönlendirir. Her thread kendi hücresinde CAS yaptığı için başka thread'lerle çakışma ihtimali düşer. `sum()`, tüm hücrelerin değerlerini toplayarak toplam sayıyı hesaplar; bu toplama işlemi atomik bir anlık görüntü değildir, toplama sırasında bir thread hala yazıyor olabilir.

## AtomicLong ile aynı senaryo

```java
AtomicLong atomicHits = new AtomicLong();

Runnable worker = () -> {
    for (int i = 0; i < 100_000; i++) {
        atomicHits.incrementAndGet();
    }
};
// aynı iki thread, aynı 100_000 döngü

System.out.println(atomicHits.get()); // 200000, ama çok daha fazla CAS çakışması ve yeniden deneme ile
```

Sonuç her iki durumda da doğrudur, fark performansta ortaya çıkar: `AtomicLong` tüm thread'leri aynı `value` alanı üzerinde yarıştırır, `LongAdder` ise yazma yükünü hücrelere bölerek yüksek thread sayısında çok daha az CAS başarısızlığı üretir. Buna karşılık `getAndIncrement` gibi "güncel değeri oku ve artır" garantisi `LongAdder`'da yoktur; `sum()` sadece anlık bir yaklaşık toplamdır.

## Ne zaman işe yarar

- Sadece toplama/sayma amaçlı, çok sayıda thread'in sürekli yazdığı sayaçlarda (istek sayacı, metrik toplama) `LongAdder` tercih edilmelidir.
- Güncellenen değerin anında, tutarlı okunması gerekiyorsa (örneğin `compareAndSet` ile koşullu güncelleme veya versiyon numarası) `AtomicLong` kullanılmalıdır; `LongAdder` bu tür atomik okuma-karşılaştırma işlemlerini desteklemez.
- Düşük çekişme altında (az thread, seyrek güncelleme) iki sınıf arasında anlamlı bir fark yoktur; `LongAdder`'ın avantajı yalnızca yüksek eşzamanlılıkta ortaya çıkar.

## Denemek için

```
jshell> var a = new java.util.concurrent.atomic.LongAdder()
jshell> a.increment(); a.increment(); a.add(5)
jshell> a.sum()
jshell> var al = new java.util.concurrent.atomic.AtomicLong(); al.incrementAndGet()
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/LongAdder.html]
