# StringBuilder.reverse() ve Surrogate Tuzağı

`StringBuilder.reverse()`, bir metni tersine çevirmenin en pratik yoludur, ama Java `String`'leri UTF-16 kod birimleriyle (`char`) temsil eder; bazı Unicode karakterler tek bir `char`'a sığmaz. `reverse()` bu gerçeği kısmen bilir: gerçek *surrogate pair*'leri (örneğin emoji) bozmadan taşır, ama bir harfe eklenmiş *combining* işaretleri (aksan, nokta gibi birleşen karakterler) aynı özenle korumaz ve sırasını değiştirerek anlamı bozar.

## Surrogate pair doğru taşınır

```java
StringBuilder sb = new StringBuilder("a😀b"); // a + 😀 (U+1F600) + b
System.out.println(sb.reverse()); // "b😀a"
```

`😀` tek bir Unicode kod noktasını (😀) temsil eden bir *high-low surrogate* çiftidir. `reverse()` javadoc'unda açıkça belirtildiği gibi, bu çiftleri tek birim olarak ele alır ve yüksek/alçak surrogate sırasını asla ters çevirmez; sonuçta emoji bozulmadan `b` ile `a` arasına taşınır.

## Combining karakter bozulur

```java
StringBuilder sb2 = new StringBuilder("é"); // e + birleşen ince aksan (´)
System.out.println(sb2.reverse()); // "´e" gibi görünen, anlamsız bir dizi
```

`́` kendi başına bağımsız bir kod noktasıdır (`COMBINING ACUTE ACCENT`), surrogate pair değildir; `reverse()` onu tabanıyla (`e`) birlikte bir birim saymaz, iki `char`'ı ayrı ayrı tersine çevirir. Sonuçta aksan karakteri taban harften önce gelir ve görsel/anlamsal bütünlük kaybolur. Aynı sorun *zero-width joiner* ile birleşen çok parçalı emoji dizilerinde de ortaya çıkar.

## Dikkat edilmesi gerekenler

- `reverse()` yalnızca UTF-16 surrogate pair'leri korur; `grapheme cluster` (taban karakter + birleşen işaretler) kavramından habersizdir.
- Kullanıcıdan gelen, aksanlı dillerde yazılmış veya emoji içeren metinleri tersine çevirmeden önce `java.text.Normalizer` ile `NFC`/`NFD` normalize etmek sorunu çözmez; asıl ihtiyaç kod noktası değil *grapheme* bazlı bir tersine çevirme algoritmasıdır.
- Salt ASCII metinlerde bu tuzak hiç görünmez, bu da testlerde fark edilmemesini kolaylaştırır.

## Denemek için

```
jshell> new StringBuilder("a😀b").reverse().toString().codePoints().forEach(c -> System.out.println(Integer.toHexString(c)))
jshell> new StringBuilder("é").reverse().toString().codePoints().forEach(c -> System.out.println(Integer.toHexString(c)))
```

İlk komutta `62`, `1f600`, `61` sırasını (b, 😀, a) görürsünüz; kod noktası bozulmamıştır. İkinci komutta ise `301`, `65` sırasını (önce aksan, sonra e) görürsünüz; taban harfle işaretin yeri değişmiştir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html#reverse()]
