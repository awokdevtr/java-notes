# Arayüzlerde private metotlar

Java 8 ile gelen `default` metotlar arayüzlere davranış eklemeyi mümkün kıldı, ama birden fazla `default` metot aynı yardımcı mantığı tekrar etmeye başladığında bu kodu paylaşacak bir yer yoktu; yardımcı mantığı `default` olarak yazmak onu arayüzün herkese açık sözleşmesine dahil ederdi. Java 9, `private` metotlara arayüz içinde izin vererek bu tekrarı implementasyon detayı olarak gizlemenin yolunu açtı.

## Temel kullanım

```java
interface Fatura {

    double tutar();

    default String ozet() {
        return "Tutar: " + formatla(tutar());
    }

    default String detayliOzet() {
        return "Fatura - " + formatla(tutar()) + " (KDV dahil)";
    }

    private String formatla(double deger) {
        return String.format("%.2f TL", deger);
    }
}
```

`formatla`, `ozet` ve `detayliOzet` metotlarının paylaştığı biçimlendirme mantığını barındırır ama arayüzü implement eden sınıflar tarafından çağrılamaz, override edilemez ve implementasyon zorunlu kılınmaz. Bu, soyut sınıflardaki `private` yardımcı metotların arayüzlerdeki karşılığıdır: davranışı merkezileştirir, sözleşmeyi büyütmez.

## static private metotlar

```java
interface SayiYardimcisi {

    static int kareAl(int x) {
        return carp(x, x);
    }

    static int kupAl(int x) {
        return carp(carp(x, x), x);
    }

    private static int carp(int a, int b) {
        return a * b;
    }
}
```

`private` metotlar `static` da olabilir; bu durumda yalnızca arayüzün diğer `static` metotlarından çağrılabilir, bir örnek (`this`) gerektirmez. Kural nettir: `private` bir instance metodu sadece `default`/`private` instance metotlarından, `private static` bir metot ise sadece `static` metotlardan çağrılabilir.

## Dikkat edilmesi gerekenler

- `private` arayüz metotları `abstract` olamaz ve gövdesiz tanımlanamaz; soyut metotlarla karıştırılmamalı.
- Alt sınıflar bu metotları ne görür ne override edebilir; derleme zamanında arayüz dışından erişim denemesi hata verir.
- Amaç kod tekrarını azaltmak, API yüzeyini küçültmektir; arayüzü sınıf gibi kullanıp karmaşık iç durum tutmaya başlamak kötü bir işarettir.

## Denemek için

```
jshell> interface F { default String ozet(double t) { return go(t); } private String go(double t) { return "~" + t; } }
jshell> class X implements F {}
jshell> new X().ozet(9.5)
```

`go` metodunu `X` üzerinden doğrudan çağırmayı denerseniz (`new X().go(9.5)`), derleyici bu metodun görünür olmadığını belirten bir hata verir; `private` erişim arayüz sınırının dışına hiç çıkmaz.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html#jls-9.4]
