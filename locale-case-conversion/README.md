# Locale'e bagli buyuk/kucuk harf donusumu

`String.toUpperCase()` ve `String.toLowerCase()` parametresiz cagrildiginda JVM'in calistigi makinenin varsayilan `Locale`'ini kullanir. Bu, ayni kodun bir sunucuda dogru calisip baska bir sunucuda (farkli isletim sistemi dil ayariyla) sessizce yanlis sonuc uretmesine yol acabilir. En bilinen ornek Turkce `Locale`'dir: `i` harfinin buyutulmesi beklenen `I` degil, noktali `İ` (U+0130) olur.

## Sorunu goren kod

```java
String input = "istanbul";
String upper = input.toUpperCase();
System.out.println(upper.equals("ISTANBUL"));
```

Varsayilan `Locale` Turkce (`tr` veya `tr-TR`) olan bir makinede bu kod `false` basar, cunku `upper` degeri `"İSTANBUL"` olur (bastaki `İ`, Latin `I` degil `U+0130 LATIN CAPITAL LETTER I WITH DOT ABOVE`'dir). Ayni sekilde `"TITLE".toLowerCase()` Turkce ortamda `"tıtle"` dondurur; `I` harfi noktasiz `ı` olur. Bu kod ingilizce varsayilan locale'li bir makinede test edilip gecerse, uretimde Turkce locale'li bir sunucuda calisinca `equals`, `Map` anahtari karsilastirmasi ya da sabit string kontrolu beklenmedik sekilde kirilir.

## Guvenli kullanim

```java
String input = "istanbul";
String upper = input.toUpperCase(java.util.Locale.ROOT);
System.out.println(upper.equals("ISTANBUL"));
```

`Locale.ROOT`, hicbir dile ozel kural icermeyen, tamamen yalin bir `Locale` degeridir. Protokol anahtar kelimeleri, dosya uzantilari, HTTP header'lari gibi insan diline bagli olmayan degerleri karsilastirmadan once donusturmek icin her zaman `toUpperCase(Locale.ROOT)` veya `toLowerCase(Locale.ROOT)` kullanilmalidir. Eger donusum gercekten kullanicinin goruntuleme dilini yansitmasi gerekiyorsa (ornegin arayuzde baslik gostermek icin) o zaman bilerek kullanicinin `Locale`'i verilir; ayrim, "bu metin makineler icin mi yoksa insan icin mi" sorusuna gore yapilir.

## Dikkat edilmesi gerekenler

- `equalsIgnoreCase` dogrudan locale etkisine girmez ama kendi ic implementasyonu yine karakter bazli buyuk/kucuk harf kurallarina dayanir; kritik karsilastirmalarda yine de `Locale.ROOT` ile normalize edip `equals` kullanmak daha ongorulebilirdir.
- `String.format` de parametresiz cagrildiginda varsayilan `Locale`'i kullanir; sayisal ayirici (`,` vs `.`) farkli locale'lerde degisebilir, bu da ayni sinifta benzer bir tuzaktir.
- Sabit (hardcoded) anahtar kelimeler, enum adlari, dosya uzantilari gibi "protokol" degerlerini karsilastirirken asla varsayilan locale'e guvenmeyin.

## Denemek icin

```
jshell> java.util.Locale.setDefault(new java.util.Locale("tr", "TR"))
jshell> "istanbul".toUpperCase()
jshell> "istanbul".toUpperCase(java.util.Locale.ROOT)
```

Ilk cagri `"İSTANBUL"` (noktali İ) dondururken ikincisi `"ISTANBUL"` dondurur; aradaki fark tek basina `equals` kontrollerini kirabilecek kadar onemlidir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#toUpperCase(java.util.Locale)]
