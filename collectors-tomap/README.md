# Collectors.toMap ile stream'den Map oluşturma

Bir listeyi anahtar-değer eşlemesine dönüştürmek için `Collectors.toMap` sık kullanılan bir terminal operasyondur, ama iki argümanlı hali çakışan anahtarlarla karşılaşınca sessizce devam etmez: akışı çalışma zamanında `IllegalStateException` ile durdurur. Bu tuzak genelde veri setinde tekrar eden anahtar olmayacağı varsayımıyla üretime sızar.

## Çakışan anahtar sorunu

```java
record Calisan(String departman, String ad) {}

List<Calisan> calisanlar = List.of(
    new Calisan("IK", "Ayşe"),
    new Calisan("IK", "Mehmet"),
    new Calisan("Muhasebe", "Ali")
);

Map<String, String> departmanToAd = calisanlar.stream()
    .collect(Collectors.toMap(Calisan::departman, Calisan::ad));
```

İki `Calisan` kaydı da `"IK"` anahtarını ürettiği için bu çağrı `IllegalStateException: Duplicate key IK` fırlatır. İki argümanlı `toMap(keyMapper, valueMapper)` üçüncü bir `merge` fonksiyonu almadığından, çakışmayı çözecek bir strateji tanımlanmamıştır; `Collectors.toMap` bu durumu `Map.merge`'e devretmek yerine doğrudan istisna olarak işaretler.

## merge fonksiyonu ile çakışmayı çözme

```java
Map<String, String> departmanToAdlar = calisanlar.stream()
    .collect(Collectors.toMap(
        Calisan::departman,
        Calisan::ad,
        (eski, yeni) -> eski + ", " + yeni
    ));
```

Üç argümanlı overload'daki `BinaryOperator<V>`, aynı anahtar ikinci kez üretildiğinde önceki değer (`eski`) ile yeni değeri (`yeni`) alır ve dönüş değeri o anahtarın yeni değeri olur; burada iki ismi virgülle birleştirerek çakışmayı veri kaybı yaratmadan çözüyoruz. İhtiyaca göre `(eski, yeni) -> eski` ile ilk görüleni korumak veya `(eski, yeni) -> yeni` ile sonuncuyu geçerli saymak da aynı yerde tanımlanabilir.

## Dikkat edilmesi gerekenler

- Varsayılan dönen `Map`, özellikle belirtilmediği sürece bir `HashMap`'tir; eleman sırası garanti edilmez. Sıra önemliyse dört argümanlı overload ile `LinkedHashMap::new` veya `TreeMap::new` map supplier olarak geçilebilir.
- `keyMapper` veya `valueMapper` `null` üretirse `toMap`, `HashMap.merge`'in `null` değeri kabul etmemesi nedeniyle `NullPointerException` fırlatır; bu durum çakışma olmasa da ortaya çıkar.
- Paralel stream'lerde `merge` fonksiyonunun çağrılma sırası deterministik değildir; bu yüzden fonksiyonun yalnızca o anki iki değere bakan, sıraya bağımlı olmayan (örneğin ekleme değil toplama gibi) bir mantık izlemesi gerekir.

## Denemek için

```
jshell> import java.util.stream.Collectors
jshell> import java.util.List
jshell> List.of("elma", "armut", "elma").stream().collect(Collectors.toMap(s -> s, String::length))
```

Son satır `Duplicate key elma` ile patlar; üçüncü argüman olarak `(a, b) -> a` eklenip tekrar denendiğinde çakışma sessizce çözülür.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function,java.util.function.BinaryOperator)]
