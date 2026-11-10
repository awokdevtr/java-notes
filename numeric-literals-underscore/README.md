# Sayısal Literallerde Alt Çizgi

Uzun bir sayısal literal -- bir kredi kartı numarası, bir byte sayısı, bir binary maske -- göz tarafından gruplanmadan okunduğunda basamak sayısını yanlış saymak kolaydır. Java 7 ile gelen alt çizgi (`_`) ayracı, derleyiciye hiçbir anlam ifade etmeden sadece okunabilirlik için basamakları gruplama imkanı verir; literal derlendiğinde alt çizgiler tamamen silinir, sayının değerinde hiçbir etkisi olmaz.

## Basamak gruplama

```java
int kartNumarasi = 1234_5678_9012_3456;
long dosyaBoyutu = 10_000_000_000L;
int ikilikMaske = 0b1010_0101_1111_0000;
int onAltilikRenk = 0xFF_EC_DE_5A;
```

Her örnekte alt çizgiler sadece görsel bir ayraç görevi görür; `1234_5678_9012_3456` ile `1234567890123456` derleyici için birebir aynı değerdir. `0b` ve `0x` önekiyle yazılan `long`/`int` literallerde de aynı kural geçerlidir, bu da özellikle bit maskeleri ve binary sabitlerde dört'erli veya sekiz'erli gruplamayla okunabilirliği artırır.

## Yerleştirme kuralları

```java
// double pi = 3._14;        // derleme hatası: ondalık noktaya bitişik
// long x = 100_L;           // derleme hatası: L son ekine bitişik
// int y = _1000;            // derleme hatası: literalin başında
int z = 1_000;               // geçerli
```

Alt çizgi sadece birbirini takip eden iki basamak arasına yerleştirilebilir; literalin en başına, en sonuna, ondalık noktanın (`.`) hemen öncesine/sonrasına, `L`/`F`/`D` gibi tür son eklerinin hemen öncesine ya da `0x`/`0b` önekinin hemen sonrasına konulamaz. Bu kurallardan biri ihlal edildiğinde derleyici hatayı derleme zamanında yakalar; dolayısıyla yanlış yerleştirilmiş bir alt çizgi çalışma zamanında sessizce yanlış bir değere dönüşmez.

## Dikkat edilmesi gerekenler

- Alt çizgi `String.valueOf`, `Integer.parseInt` gibi metotlara verilen string girdilerde desteklenmez; bu sadece kaynak koddaki literal sözdizimine özgü bir derleyici kolaylığıdır.
- Ardışık birden fazla alt çizgi (`1__000`) geçerlidir ama okunabilirliği azaltır; tek alt çizgi kullanmak idiomatiktir.
- Grup boyutunu tutarlı tutmak (örn. hep üçerli veya hep dörderli) kodun gerçek anlamını (ondalık mı, byte mı, bit maskesi mi) daha açık gösterir.

## Denemek için

```
jshell> int n = 1_000_000
jshell> long big = 9_223_372_036_854_775_807L
jshell> int mask = 0b1111_0000_1010_0101
jshell> n
```

`n` değişkeninin `1000000` olarak yazdırıldığını, yani alt çizgilerin değere hiç karışmadığını gözlemleyin.

[Kaynak: https://docs.oracle.com/javase/8/docs/technotes/guides/language/underscores-literals.html]
