# ThreadLocalRandom vs Random

Birden fazla thread aynı `java.util.Random` örneğini paylaştığında, her `nextInt()` çağrısı iç `seed` alanını güncellemek için `AtomicLong.compareAndSet` kullanır. Çekişme yüksek olduğunda bu CAS döngüsü sürekli başarısız olup yeniden denenir ve paylaşılan `Random` nesnesi performans darboğazına dönüşür. `ThreadLocalRandom`, her thread'e kendi izole üretecini vererek bu çekişmeyi tamamen ortadan kaldırır.

## Paylaşılan Random'ın çekişme sorunu

```java
static final Random SHARED = new Random();

Runnable task = () -> {
    for (int i = 0; i < 1_000_000; i++) {
        SHARED.nextInt(100);
    }
};
```

Çok sayıda thread bu görevi aynı anda çalıştırdığında, hepsi `SHARED` örneğinin tek bir `seed` alanı üzerinde CAS ile yarışır. Thread sayısı arttıkça verim artmaz, tam tersine CAS yeniden deneme oranı yükseldiği için düşer. `Random` thread-safe'tir ama bu güvenlik, paylaşılan durumun senkronizasyon maliyetiyle gelir.

## ThreadLocalRandom ile izole üretim

```java
Runnable task = () -> {
    ThreadLocalRandom rnd = ThreadLocalRandom.current();
    for (int i = 0; i < 1_000_000; i++) {
        rnd.nextInt(100);
    }
};
```

`ThreadLocalRandom.current()`, çağıran thread'e özel bir üretecin referansını döndürür; bu üreteç `ThreadLocal` benzeri bir mekanizmayla her thread için ayrı tutulur ve hiçbir CAS veya kilit gerekmez. `nextInt(origin, bound)` gibi aralık belirten overload'lar da modulo işlemindeki yanlılığı (`bias`) önleyecek şekilde tasarlanmıştır.

## Dikkat edilmesi gerekenler

- `ThreadLocalRandom` örneği bir alanda saklanıp başka bir thread'e geçirilmemelidir; sözleşmesi sadece `current()` çağrısını yapan thread içinde geçerlidir.
- `setSeed()` çağrısı `UnsupportedOperationException` fırlatır, çünkü üretecin tohumu thread'e özgü dahili durumdan türetilir ve dışarıdan sabitlenemez; bu da testlerde deterministik sonuç almayı zorlaştırır.
- Kriptografik güvenlik gerekiyorsa (`token`, şifre vb.) bu sınıf yerine `java.security.SecureRandom` kullanılmalıdır.

## Denemek için

```
jshell> import java.util.concurrent.ThreadLocalRandom
jshell> var r = ThreadLocalRandom.current()
jshell> r.nextInt(1, 7)
```

`nextInt(1, 7)` çağrısını birkaç kez tekrarlayarak 1 ile 6 arasında (üst sınır hariç) değerler aldığınızı, her çağrının bağımsız ve kilitsiz çalıştığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadLocalRandom.html]
