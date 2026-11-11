# Cleaner API

`Object.finalize()` uzun süredir kaynak temizliği (dosya tanıtıcısı, native bellek, soket) için kullanılıyordu ama GC'nin ne zaman çalışacağını garanti etmemesi, finalizer zincirlerinin performansı düşürmesi ve hatalı bir `finalize()`'ın nesneyi "diriltebilmesi" yüzünden Java 9'da kullanımdan kaldırıldı (deprecated for removal). `java.lang.ref.Cleaner`, aynı ihtiyacı -- nesne artık erişilemez olduğunda bir temizlik eylemi tetiklemek -- daha güvenli ve öngörülebilir bir API üzerinden karşılar.

## Temizlik eylemini kaydetme

```java
import java.lang.ref.Cleaner;

class NativeResource implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        @Override public void run() {
            System.out.println("native kaynak serbest bırakıldı");
        }
    }

    private final State state = new State();
    private final Cleaner.Cleanable cleanable;

    NativeResource() {
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

`cleaner.register(obj, action)`, `obj` çöpe çıktığında (phantom reachable olduğunda) çalıştırılacak `action`'ı ayrı bir `Runnable` olarak kaydeder; `action` kesinlikle `obj`'ye referans tutmamalıdır, yoksa nesne hiçbir zaman toplanamaz. `close()` çağrısı `cleanable.clean()` ile eylemi hemen ve sadece bir kez tetikler; `close()` hiç çağrılmazsa GC nesneyi topladığında eylem otomatik olarak (ama ne zaman olacağı garanti edilmeden) arka planda çalışır.

## finalize() ile karşılaştırma

```java
// Eski, önerilmeyen yaklaşım:
@Override
protected void finalize() throws Throwable {
    // native kaynağı serbest bırak
}
```

`finalize()` nesnenin kendi sınıfında tanımlanır ve GC thread'i tarafından çağrılır; bu nesnenin `this` referansını dışarı sızdırma riski taşır ve nesneyi yeniden canlandırabilir. `Cleaner` ise temizlik mantığını ayrı bir `State` nesnesine taşıyarak bu sızıntıyı yapısal olarak engeller, çünkü `state` kayıt sırasında `this`'e değil kendi alanlarına referans tutar.

## Dikkat edilmesi gerekenler

- `Cleaner` asla deterministik kaynak yönetiminin yerine geçmez; esas kapatma mekanizması `try-with-resources` ile `close()` olmalı, `Cleaner` sadece unutulan `close()` çağrıları için bir güvenlik ağıdır.
- Temizlik eylemi (`Runnable`) kayıtlı nesneye (`this`'e) hiçbir şekilde referans tutmamalı, aksi halde nesne asla phantom reachable olmaz ve eylem tetiklenmez.
- `cleaner.register()` çağrısı her nesne için ayrı bir arka plan thread kaynağı tüketir; `Cleaner.create()` tek bir paylaşılan `Cleaner` örneği olarak tutulup tüm nesneler arasında yeniden kullanılmalıdır.

## Denemek için

```
jshell> var cleaner = java.lang.ref.Cleaner.create()
jshell> var obj = new Object()
jshell> var cleanable = cleaner.register(obj, () -> System.out.println("temizlendi"))
jshell> cleanable.clean()
```

`clean()` çağrıldığı anda "temizlendi" çıktısının hemen basıldığını, tekrar çağrıldığında ise hiçbir şey yapmadığını (idempotent) gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Cleaner.html]
