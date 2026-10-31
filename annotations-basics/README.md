# Temel Annotation Kullanımı

`@Override` ya da `@Deprecated` gibi hazır annotation'ları herkes kullanır, ama bir annotation'ın derleme zamanında mı yoksa çalışma zamanında mı görünür olacağını belirleyen şey genelde gözden kaçar. Kendi annotation'ınızı tanımlayıp runtime'da okumak istediğinizde, bu görünürlüğü doğru ayarlamazsanız reflection ile hiçbir şey bulamazsınız.

## Özel annotation tanımlamak

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Experimental {
    String reason() default "henüz kararlı değil";
}
```

`@interface` ile tanımlanan bu yapı, meta-annotation'larla kendi davranışını belirtir. `@Target(ElementType.METHOD)`, `Experimental`'ın sadece metotlara eklenebileceğini söyler; bir sınıfa veya alana eklenmeye çalışılırsa derleme hatası alınır. `@Retention(RetentionPolicy.RUNTIME)` ise annotation bilgisinin `.class` dosyasına yazılıp JVM tarafından çalışma zamanında da erişilebilir kalacağını belirtir; varsayılan politika olan `CLASS` ise bilgiyi bytecode'a yazar ama reflection'a kapatır, `SOURCE` ise bilgiyi derlemeden sonra tamamen siler.

## Reflection ile okumak

```java
class Service {
    @Experimental(reason = "performans testi bitmedi")
    void riskliMetot() {}
}

Method m = Service.class.getDeclaredMethod("riskliMetot");
if (m.isAnnotationPresent(Experimental.class)) {
    Experimental meta = m.getAnnotation(Experimental.class);
    System.out.println(meta.reason());
}
```

`RetentionPolicy.RUNTIME` seçilmediyse `getAnnotation` her zaman `null` döner, çünkü bilgi bytecode'da hiç bulunmaz; bu en sık yapılan hatadır. `isAnnotationPresent` ve `getAnnotation`, `java.lang.reflect.Method`, `Class` veya `Field` üzerinde çalışır ve annotation elemanlarına (`reason()` gibi) doğrudan metot çağrısı gibi erişim sağlar.

## Ne zaman işe yarar

- Test çerçeveleri (`@Test`, `@BeforeEach`) ve bağımlılık enjeksiyonu (`@Inject`, `@Autowired`) gibi araçlar, kodu değiştirmeden ona üst düzey anlam eklemek için bu mekanizmayı kullanır.
- Derleme zamanı kontrolü yeterli olduğunda (`@Override` gibi) `RUNTIME` yerine `SOURCE` veya `CLASS` seçmek, bilgiyi gereksiz yere çalışma zamanına taşımaz.
- `@Retention` belirtilmezse varsayılan `CLASS` olur; reflection tabanlı bir araç yazıyorsanız bunu açıkça `RUNTIME` yapmayı unutmak sessizce başarısız olur.

## Denemek için

```
jshell> import java.lang.annotation.*;
jshell> @Retention(RetentionPolicy.RUNTIME) @interface Not { String deger() default ""; }
jshell> class X { @Not(deger="test") void m() {} }
jshell> X.class.getDeclaredMethod("m").getAnnotation(Not.class).deger()
```

Son satır `"test"` döndürür; `@Retention`'ı `RetentionPolicy.SOURCE` olarak değiştirip tekrar denediğinizde `getAnnotation` çağrısı `null` döner, çünkü annotation bilgisi artık bytecode'da bulunmaz.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/annotations/index.html]
