# StackWalker ile Çağrı Yığınını Gezme

Eskiden çağrı yığınını (call stack) incelemek için `Thread.currentThread().getStackTrace()` veya `new Throwable().getStackTrace()` kullanılırdı; bu yöntemler tüm yığını eagerly bir `StackTraceElement[]` dizisine kopyalar, lambda ve reflection gibi "gizli" çerçeveleri (hidden frames) ayıklamayı zorlaştırır ve sadece ilk birkaç çerçeveye ihtiyaç duyulan durumlarda bile gereksiz maliyete yol açar. Java 9 ile gelen `java.lang.StackWalker`, yığını lazy (tembel) bir `Stream<StackFrame>` olarak sunar.

## Temel Kullanım

```java
StackWalker walker = StackWalker.getInstance();
Optional<String> caller = walker.walk(frames ->
    frames.skip(1)
          .findFirst()
          .map(StackWalker.StackFrame::getClassName));
caller.ifPresent(System.out::println);
```

`walk` metodu, bir `Stream<StackFrame>` alıp bir sonuç üreten fonksiyon bekler; stream yalnızca `walk` çağrısı sürerken geçerlidir, çünkü arkasındaki yığın anlık görüntüsü callback'ten sonra serbest bırakılır. `skip(1)` ile çalışan metodun kendi çerçevesi atlanır ve doğrudan çağıranın sınıf adına ulaşılır.

## Çağıran Sınıfı Doğrudan Almak

```java
StackWalker walker = StackWalker.getInstance(StackWalker.Option.RETAIN_CLASS_REFERENCE);
Class<?> callerClass = walker.getCallerClass();
System.out.println(callerClass.getName());
```

`RETAIN_CLASS_REFERENCE` seçeneği verilmeden `getCallerClass()` çağrılırsa `UnsupportedOperationException` fırlar, çünkü sınıf referansını canlı tutmak class-loader'ın garbage collection'ını geciktirebilecek ek bir bellek maliyeti getirir ve bu nedenle varsayılan olarak açık değildir.

## Dikkat edilmesi gerekenler

- `walk` içine verilen fonksiyonun döndürdüğü stream, liste veya `StackFrame` referansı, `walk` çağrısı bittikten sonra güvenle kullanılamaz; ihtiyaç duyulan veri callback içinde `toList()`/`collect` ile somut bir koleksiyona çekilmelidir.
- `StackWalker.Option.SHOW_HIDDEN_FRAMES` verilmediği sürece lambda, reflection ve sentetik köprü metotları yığından otomatik olarak gizlenir; bu, `getStackTrace()`'in her çerçeveyi gösteren davranışından farklıdır.
- `StackWalker`, güvenlik yöneticisi (varsa) ve modül sınırları açısından `getStackTrace()`'ten daha sıkı kontrol edilir; bazı çerçeveler çağıranın erişim izni olmadan görünmez.

## Denemek için

```
jshell> import java.lang.StackWalker.StackFrame
jshell> StackWalker.getInstance().walk(s -> s.map(StackFrame::toString).collect(java.util.stream.Collectors.joining("\n")))
```

Bu komut, o anki jshell oturumunun çağrı yığınını satır satır yazdırır; her satır ilgili çerçevenin sınıf ve metot bilgisini içerir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.html]
