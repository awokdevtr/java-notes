# İşaretli ve İşaretsiz Sağa Kaydırma (`>>` ve `>>>`)

Negatif bir sayıyı bit düzeyinde sağa kaydırmak gerektiğinde `>>` ve `>>>` operatörleri birbirine çok benzer görünür, ama tamamen farklı sonuçlar üretir. `>>` işaret bitini korurken `>>>` onu da sıfırla doldurur; bu farkı atlamak, özellikle hash hesaplamaları veya bit maskesi işlemlerinde beklenmedik negatif değerlerin ortaya çıkmasına yol açar.

## İşaret biti nasıl korunur, nasıl kaybolur

```java
int negatif = -8;                 // 11111111 11111111 11111111 11111000
System.out.println(negatif >> 1);  // -4  (isaretli: en soldaki bit kopyalanir)
System.out.println(negatif >>> 1); // 2147483644  (isaretsiz: en soldan 0 girer)
```

`>>` (`signed right shift`), en soldaki işaret bitini kopyalayarak kaydırır; bu yüzden negatif bir sayıyı `>>` ile kaydırmak sonucu negatif tutar ve matematiksel olarak `2`'ye bölmeye (aşağı yuvarlayarak) eşdeğerdir. `>>>` (`unsigned right shift`) ise en soldan her zaman `0` sokar, dolayısıyla negatif bir `int` anında büyük bir pozitif sayıya dönüşür. Pozitif sayılarda iki operatör aynı sonucu verir çünkü işaret biti zaten `0`'dır.

## `hashCode` içindeki klasik kullanım

```java
int h = someHash;
int mixed = h ^ (h >>> 16);
```

`HashMap` gibi sınıfların iç `hash()` fonksiyonlarında `>>>` tercih edilir çünkü amaç sayıyı işaretli bir değer gibi değil, ham bit deseni gibi karıştırmaktır. Burada `>>` kullanılsaydı negatif `hashCode()` değerlerinde işaret biti tekrarlanarak üst bitler "kaybolmaz" ama alt bitlerle karışım bozulurdu; `>>>` tüm 32 biti eşit şekilde devreye katar.

## Dikkat edilmesi gerekenler

- `long` için kaydırma miktarı `64` ile, `int` için `32` ile mod alınır: `1 >> 33` derleyicide `1 >> 1` ile aynıdır, hata vermez.
- `>>>` operatörünün `byte` ve `short` üzerinde kullanımı önce `int`'e genişletme (`promotion`) yapar, bu yüzden `(byte) -8 >>> 1` beklenen küçük pozitif değeri değil büyük bir `int` sonucu verir.
- `<<` (sola kaydırma) için işaretli/işaretsiz ayrımı yoktur; fark yalnızca sağa kaydırmada ortaya çıkar.

## Denemek için

```
jshell> -8 >> 1
jshell> -8 >>> 1
jshell> Integer.toBinaryString(-8 >>> 1)
```

İlk satır `-4` döner, ikinci satır `2147483644` gibi büyük bir pozitif sayı döner; üçüncü satırda ikilik tabanı yazdırarak en soldaki bitin `>>>` sonucunda neden `0` olduğunu doğrudan görebilirsiniz.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html]
