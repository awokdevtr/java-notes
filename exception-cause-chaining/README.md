# Exception Cause Chaining

Düşük seviyeli bir hatayı (örn. `SQLException`) yakalayıp daha anlamlı bir uygulama istisnasına (`DataAccessException`) çevirmek yaygın bir pratiktir. Ancak orijinal istisnayı bilgisiz şekilde `throw new DataAccessException("kayıt bulunamadı")` ile sarmak, asıl kök nedeni (hangi SQL hatası, hangi stack trace) tamamen kaybeder. `Throwable`'ın **cause chaining** mekanizması bu bilgiyi korur.

## `initCause` ve cause alan constructor'lar

```java
try {
    Integer.parseInt("abc");
} catch (NumberFormatException e) {
    throw new IllegalStateException("Konfigürasyon değeri geçersiz", e);
}
```

`IllegalStateException(String, Throwable)` gibi iki parametreli constructor'lar, verilen `e`'yi dahili `cause` alanına atar. Alternatif olarak tek parametreli bir constructor çağrıldıysa sonradan `exception.initCause(e)` ile de aynı ilişki kurulabilir — ama `initCause` bir istisna üzerinde yalnızca **bir kez** çağrılabilir, ikinci çağrıda `IllegalStateException` fırlatılır. Cause atanmadıysa `getCause()` `null` döner.

## Zincirin stack trace üzerinde görünmesi

```java
try {
    throw new IllegalStateException("dış hata", new NumberFormatException("iç hata"));
} catch (IllegalStateException e) {
    e.printStackTrace();
}
// Çıktı: IllegalStateException: dış hata
//   ...
// Caused by: NumberFormatException: iç hata
//   ...
```

`printStackTrace()` cause zincirini `Caused by:` satırlarıyla otomatik olarak yazdırır; `getCause()` ile programatik olarak da gezilebilir, zincirin kökü `getCause()` `null` dönene kadar takip edilerek bulunur. Bu, "exception wrapping" yapılan kodlarda hata ayıklamayı kurtaran temel özelliktir — sarmalama olmadan kök neden kalıcı olarak kaybolur.

## Dikkat edilmesi gerekenler

- Bir istisnayı yeniden fırlatırken cause'u elden kaçırmak (`throw new X(msg)` yerine `throw new X(msg, e)`) en sık yapılan hatadır ve hata ayıklamayı ciddi şekilde zorlaştırır.
- `getSuppressed()` farklı bir mekanizmadır: `try-with-resources` içinde `close()` sırasında oluşan ek istisnaları tutar, `getCause()` ile karıştırılmamalıdır.
- Serileştirilen istisnalarda cause zinciri de serileştirilir; zincirdeki her sınıfın `Serializable` olması gerekir.

## Denemek için

```
jshell> try { throw new RuntimeException("dış", new IllegalArgumentException("iç")); } catch (RuntimeException e) { System.out.println(e.getCause()); }
```

Çıktı `java.lang.IllegalArgumentException: iç` olur — dışarıdaki istisna yakalanırken içerideki kök nedene doğrudan erişilebildiği görülür.

[Kaynak: https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html#getCause--]
