# VarHandle ile alan ve dizi elemanlarına kontrollü erişim

Bir alanı bazen sıradan bir okuma/yazma ile, bazen `volatile` garantisiyle, bazen de atomik `compareAndSet` ile değiştirmek gerekir. Bunu sağlamanın klasik yolu alanı `volatile` yapmak veya onu bir `AtomicInteger` gibi sarmalayıcı bir nesneye taşımaktır; ama bu, her erişimde aynı (en güçlü) bellek garantisini zorunlu kılar ya da ekstra bir nesne tahsisi gerektirir. `java.lang.invoke.VarHandle`, tek bir alan üzerinde erişim başına farklı bellek semantiği seçmeyi, sarmalayıcı nesneye ihtiyaç duymadan sağlar.

## CAS ile bir alanı atomik güncelleme

```java
import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;

class Counter {
    private volatile int value;

    private static final VarHandle VALUE;
    static {
        try {
            VALUE = MethodHandles.lookup()
                .findVarHandle(Counter.class, "value", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    boolean tryIncrement(int expected) {
        return VALUE.compareAndSet(this, expected, expected + 1);
    }
}
```

`MethodHandles.lookup()` çağrıldığı sınıfın erişim haklarını taşıyan bir `Lookup` nesnesi üretir; bu yüzden `findVarHandle` aynı sınıfın içinden çağrılmalıdır, aksi halde `private` alana erişim reddedilir. Üretilen `VarHandle`, `AtomicInteger` gibi ayrı bir nesneye sarmadan doğrudan `value` alanı üzerinde `compareAndSet` çalıştırır; JDK'nın kendi `Atomic*` sınıfları da içeride tam olarak bu mekanizmayı kullanır.

## Erişim modları: plain, opaque, acquire/release, volatile

```java
int[] array = new int[10];
VarHandle elements = MethodHandles.arrayElementVarHandle(int[].class);

elements.setVolatile(array, 3, 42);          // volatile yazma: tam bellek bariyeri
int plain = (int) elements.get(array, 3);    // plain okuma: hiç bariyer yok
int fresh = (int) elements.getVolatile(array, 3); // volatile okuma: güncel değeri garanti eder
```

Aynı `VarHandle` üzerinden `get`/`set` (sıradan alan erişimiyle aynı, bariyersiz), `getOpaque`/`setOpaque` (sıralama garantisi zayıf ama program sırası korunur), `getAcquire`/`setRelease` ve `getVolatile`/`setVolatile` çağrılabilir. Bu, aynı alana göre duruma uygun en ucuz erişimi seçme imkanı verir: sık okunan ama nadiren değişen bir alanda çoğu okumayı `plain` yapıp sadece kritik noktada `getVolatile` kullanmak mümkündür.

## Dikkat edilmesi gerekenler

- `findVarHandle` kontrol edilen istisnalar fırlatır (`NoSuchFieldException`, `IllegalAccessException`); genellikle statik bir blokta yakalanıp `ExceptionInInitializerError`'a çevrilir.
- `Lookup` nesnesi, üretildiği sınıfın erişim haklarını taşır; bu nesneyi başka bir sınıfa aktarmak o sınıfa kendi erişim haklarının ötesinde yetki vermiş olabilir.
- `plain` erişim modunu `volatile` gerektiren bir senaryoda kullanmak, alanı zaten `volatile` yapmanın önlediği görünürlük hatasını geri getirir; mod seçimi performans kararı kadar doğruluk kararıdır.

## Denemek için

```
jshell> import java.lang.invoke.*
jshell> class Box { public int v; }
jshell> var vh = MethodHandles.lookup().findVarHandle(Box.class, "v", int.class)
jshell> var b = new Box()
jshell> vh.compareAndSet(b, 0, 5); b.v
```

Son satır `true` ve ardından `5` döner; `compareAndSet` alanın beklenen değeri taşıdığını doğrulayıp atomik olarak günceller.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/invoke/VarHandle.html]
