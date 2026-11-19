# Base64 Encoding (java.util.Base64)

İkili veriyi (resim, şifrelenmiş anahtar, serileştirilmiş nesne) metin tabanlı bir ortamda (JSON alanı, URL, HTTP header) taşımak gerektiğinde ham `byte[]` doğrudan kullanılamaz; bazı baytlar yazdırılamaz karakterlere denk gelir ve aktarım sırasında bozulabilir. `java.util.Base64`, bu baytları yalnızca ASCII harf, rakam ve birkaç özel karakterden oluşan güvenli bir metne çevirir.

## Temel kodlama/çözme

```java
byte[] data = "merhaba".getBytes(StandardCharsets.UTF_8);

String encoded = Base64.getEncoder().encodeToString(data);
byte[] decoded = Base64.getDecoder().decode(encoded);

System.out.println(encoded);
System.out.println(new String(decoded, StandardCharsets.UTF_8));
```

`Base64.getEncoder()` ve `getDecoder()` her çağrıda yeni nesne üretmez; sınıf içinde tutulan immutable, thread-safe singleton'ları döndürür, bu yüzden her yerde güvenle paylaşılabilirler. Çıktı alfabesi `A-Z`, `a-z`, `0-9`, `+` ve `/` karakterlerinden oluşur; veri 3'ün katı olmayan bayt uzunluğundaysa sonuna `=` dolgu (`padding`) eklenir.

## URL-safe ve dolgusuz varyantlar

```java
String urlSafe = Base64.getUrlEncoder().withoutPadding().encodeToString(data);
byte[] back = Base64.getUrlDecoder().decode(urlSafe);
```

Standart alfabedeki `+` ve `/` karakterleri URL'de veya dosya adında özel anlam taşıdığından, `getUrlEncoder()` bunların yerine `-` ve `_` kullanan ayrı bir alfabe uygular. `withoutPadding()` ise sondaki `=` karakterlerini üretmez; ancak çözme tarafında da aynı dolgusuz formatın beklendiğinden emin olunmalıdır, aksi halde decoder orijinal uzunluğu yanlış hesaplayabilir.

## Dikkat edilmesi gerekenler

- Base64, bir **şifreleme** değil, bir **kodlama** yöntemidir; kodlanmış metin herkes tarafından anahtarsız çözülebilir, gizlilik sağlamaz.
- Kodlanmış veri ham veriden yaklaşık `%33` daha büyüktür (her 3 bayt, 4 karaktere dönüşür); büyük dosyalarda bu ek yükü hesaba katmak gerekir.
- `Base64.getMimeEncoder()`, çıktıyı belirli satır uzunluğunda (varsayılan 76 karakter) satır sonlarıyla böler; bu, e-posta (MIME) gibi satır uzunluğu sınırlı protokoller için tasarlanmıştır ve diğer varyantlarla karıştırılmamalıdır.

## Denemek için

```
jshell> import java.util.Base64
jshell> var enc = Base64.getEncoder().encodeToString("test123".getBytes())
jshell> enc
jshell> new String(Base64.getDecoder().decode(enc))
```

Kodlanmış metnin `=` ile bitip bitmediğine bakarak orijinal veri uzunluğunun 3'ün katı olup olmadığını tahmin edebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Base64.html]
