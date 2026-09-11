# Optional Kullanımı

Bir metot bazen anlamlı bir değer döndüremez: aranan kayıt bulunamamış ya da hesaplama sonucu yoktur. Bunu ifade etmenin klasik yolu `null` döndürmektir, ama çağıran taraf bunu kontrol etmeyi unutursa sonuç `NullPointerException`'dır. `Optional<T>`, "değer olabilir ya da olmayabilir" durumunu tip sisteminde açıkça temsil ederek bu unutkanlığı derleme zamanına yaklaştırır.

## Optional oluşturma ve okuma

```java
Optional<String> bulunan = Optional.ofNullable(kullaniciAdiGetir(id));

String ad = bulunan.orElse("bilinmiyor");
String ad2 = bulunan.orElseThrow(() -> new NoSuchElementException("kullanici yok"));
```

`Optional.of(deger)` `null` verilirse hemen istisna fırlatırken, `Optional.ofNullable(deger)` `null` durumunda boş bir `Optional` üretir; kaynağı `null` olabilecek bir yerden değer alıyorsanız ikincisini kullanmalısınız. `orElse` her zaman bir varsayılan değer verirken, `orElseThrow` değer yoksa özel bir istisna fırlatmanızı sağlar. Doğrudan `get()` çağırmak, `isPresent()` kontrolü yapılmadan kullanılırsa `NoSuchElementException` riskini `null` kontrolü kadar geri getirir; bu yüzden tercih edilmez.

## Zincirleme dönüşümler: map ve filter

```java
Optional<Kullanici> kullanici = kullaniciBul(id);

String sehir = kullanici
        .map(Kullanici::getAdres)
        .filter(a -> a.aktif())
        .map(Adres::getSehir)
        .orElse("adres yok");
```

`map`, `Optional` içindeki değer varsa ona bir dönüşüm uygular ve sonucu yine bir `Optional` içinde sarar; değer yoksa hiçbir şey yapmadan boş `Optional`'ı geçirir. `filter`, verilen koşulu sağlamayan değeri boş `Optional`'a çevirir. Bu zincirleme sayesinde ara adımlarda tekrar tekrar `null` kontrolü yazmadan, iç içe geçmiş nesne grafiklerinde güvenli gezinme yapılabilir.

## Dikkat edilmesi gerekenler

- `Optional`'ı alan (field) tipi ya da metot parametresi olarak kullanmak önerilmez; asıl amacı metot dönüş tipleridir.
- `Optional<List<T>>` yerine boş liste döndürmek genelde daha iyidir; koleksiyonlar zaten "yok" durumunu boş haliyle ifade edebilir.
- `isPresent()` ardından `get()` çağırmak yerine `map`/`orElse`/`ifPresent` gibi fonksiyonel metotları kullanmak, `Optional`'ın asıl tasarım amacına daha uygundur.

## Denemek için

```
jshell> var deger = java.util.Optional.ofNullable(null)
jshell> deger.map(String::toUpperCase).orElse("bos")
jshell> java.util.Optional.of("merhaba").map(String::toUpperCase).orElse("bos")
```

İlk `orElse` çağrısı `"bos"` döndürür çünkü `Optional` içi boştur ve `map` hiçbir şey yapmadan boş kalır; ikincisi ise `"MERHABA"` döndürür çünkü değer mevcuttur ve dönüşüm uygulanır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html]
