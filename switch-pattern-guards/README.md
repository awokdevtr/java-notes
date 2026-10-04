# switch pattern guards

`switch` içinde pattern matching ile bir nesnenin tipini kontrol edip bileşenlerine ayırmak güçlüdür, ama tip kontrolü çoğu zaman yetmez: "bu bir `Order` mi" değil, "bu bir `Order` ve `total() > 1000` mi" sorusuna cevap gerekir. Bunu `case` gövdesine `if` yazarak çözmek hem `exhaustiveness` denetimini bozar hem de pattern variable'ın kapsamını karmaşıklaştırır. Java 21, `when` anahtar kelimesiyle bir pattern'e ek koşul (`guard`) bağlamayı doğrudan dilin parçası yapar.

## when ile koşullu dal

```java
static String seviyeBelirle(Object sira) {
    return switch (sira) {
        case Siparis s when s.toplam() > 1000 -> "yuksek-oncelik";
        case Siparis s when s.toplam() > 100  -> "normal";
        case Siparis s -> "dusuk-tutar";
        default -> "bilinmeyen";
    };
}
```

Her `case`, önce `instanceof` benzeri bir tip kontrolü yapar, eşleşirse `pattern variable`'ı (`s`) bağlar ve ancak `when` sonrası koşul da `true` dönerse o dal çalışır; koşul `false` ise akış bir sonraki `case`'e düşer. Aynı tip için birden fazla `case Siparis s when ...` yazılabilir, çünkü her biri farklı bir `boolean` ifadeyle ayrışır. Dallar yukarıdan aşağı sırayla denendiği için en özel (dar) koşul en üste, genel pattern en alta yazılmalıdır — yoksa genel dal, özel olanın önüne geçip onu hiç çalıştırmaz.

## Guard'sız dal zorunluluğu

```java
sealed interface Sekil permits Daire, Kare {}
record Daire(double yaricap) implements Sekil {}
record Kare(double kenar) implements Sekil {}

static boolean buyukMu(Sekil s) {
    return switch (s) {
        case Daire d when d.yaricap() > 10 -> true;
        case Daire d -> false;
        case Kare k -> k.kenar() > 10;
    };
}
```

Derleyici `exhaustiveness` denetimini pattern'in tipine göre yapar, `when` koşuluna göre değil; bu yüzden `when` içeren bir `case` tek başına bir tipi "kapsanmış" saymaz. `Daire` için hem koşullu hem koşulsuz dal yazılmazsa derleyici eksik dal hatası verir, çünkü `when` ifadesinin çalışma zamanında `false` dönme ihtimali her zaman açık bir çıkış yolu gerektirir.

## Dikkat edilmesi gerekenler

- `when` koşulu yalnızca aynı dalda bağlanan pattern variable'ları değil, dıştaki yerel değişkenleri de kullanabilir; ama koşul `side effect` içermemelidir, çünkü hangi dalların deneneceği derleyici optimizasyonuna bağlı olabilir.
- Bir `sealed` tipte tüm alt tipler `when`'siz en az bir dalla da kapsanmadıkça `default` zorunlu hale gelir.
- Guard'lı dallar yalnızca pattern `switch`'te (tip veya `record pattern` ile) geçerlidir; klasik sabit değer eşleşmesi (`case 1 ->`) `when` kabul etmez.

## Denemek için

```
jshell> record Siparis(double toplam) {}
jshell> Object s = new Siparis(1500)
jshell> String r = switch (s) { case Siparis x when x.toplam() > 1000 -> "yuksek"; case Siparis x -> "normal"; default -> "?"; }
jshell> r
```

`r` değişkeninin `"yuksek"` olduğunu görürsünüz; `toplam` değerini `500` yapıp tekrar çalıştırırsanız ikinci dal devreye girer ve sonuç `"normal"` olur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch-statements-and-expressions.html]
