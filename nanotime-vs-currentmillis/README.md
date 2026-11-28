# nanotime-vs-currentmillis

Bir işlemin ne kadar sürdüğünü ölçmek için `System.currentTimeMillis()` çağırıp farkını almak cazip görünür, ama bu metot takvim saatini verir ve işletim sistemi NTP ile saati geriye alırsa veya kullanıcı saati değiştirirse ölçüm negatif ya da anlamsız çıkabilir. `System.nanoTime()` ise böyle bir garantiyle gelmez çünkü zaten takvimle ilgisi yoktur.

## Süre ölçümünde doğru araç

```java
long start = System.nanoTime();
doWork();
long elapsedNanos = System.nanoTime() - start;
System.out.println(elapsedNanos / 1_000_000 + " ms");
```

`nanoTime()`'ın döndürdüğü değer *mutlak* bir zaman değildir; tek anlamı olan şey, aynı JVM çalışması içinde alınan iki `nanoTime()` değerinin farkıdır. Bu yüzden değeri bir tarihe, epoch'a veya başka bir JVM'deki değere çevirmeye çalışmak anlamsızdır. Monoton artması garantili olduğundan sistem saati değişse de süre ölçümü bozulmaz; `currentTimeMillis()` bu garantiyi vermez.

## Zaman damgası gerektiğinde

```java
long epochMillis = System.currentTimeMillis();
Instant now = Instant.ofEpochMilli(epochMillis);
System.out.println(now);
```

"Bu olay ne zaman oldu" sorusuna cevap, yani bir takvim/duvar saati zaman damgası gerekiyorsa doğru metot `currentTimeMillis()`'tir (veya daha yüksek çözünürlük için `Instant.now()`). `nanoTime()`'ın başlangıç referans noktası JVM'e özgüdür, hatta gerçek saatle hiç ilişkili olmak zorunda değildir; onu bir `Instant`'a çevirmek veya loglara "zaman" olarak yazmak yanlıştır.

## Dikkat edilmesi gerekenler

- `nanoTime()` değerleri `long` taşmasına (overflow) karşı korumalı değildir; çok uzun aralıklarla alınan iki değerin farkı teorik olarak yanlış çıkabilir, ama pratikte bu aralık yıllar mertebesindedir.
- İki farklı JVM sürecinden alınan `nanoTime()` değerlerini birbiriyle karşılaştırmak anlamsızdır.
- İsmi "nano" olsa da gerçek çözünürlük işletim sistemine bağlıdır; nanosaniye hassasiyeti garanti değildir, sadece birim nanosaniyedir.

## Denemek için

```
jshell> long a = System.nanoTime()
jshell> Thread.sleep(50)
jshell> (System.nanoTime() - a) / 1_000_000.0
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/System.html#nanoTime()]
