# stringbuilder-capacity-growth

`StringBuilder`, karakterleri arka planda bir `byte[]` (JDK 9+'ta compact strings ile) veya `char[]` dizisinde tutar. `append()` çağrıları bu diziyi doldurduğunda `StringBuilder` diziyi yerinde büyütemez; `ArrayList` gibi yeni, daha büyük bir dizi ayırıp eski içeriği oraya kopyalar. Bu büyüme stratejisini bilmemek, döngü içinde çok sayıda `append()` çağrısı yaparken beklenmeyen kopyalama maliyetlerine yol açabilir.

## Varsayılan büyüme

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 20; i++) {
    sb.append("x");
}
```

Parametresiz kurucuyla oluşturulan bir `StringBuilder`'ın başlangıç kapasitesi `16` karakterdir. Kapasite dolduğunda büyüme formülü `newCapacity = oldCapacity * 2 + 2`'dir, yani `16 -> 34 -> 70 -> 142 ...` şeklinde ilerler. Her büyüme adımı `Arrays.copyOf` ile `O(n)` maliyetli bir kopyalama gerektirir; bu maliyet amortize edildiğinde tek bir `append()` çağrısı ortalama `O(1)` kalır, tıpkı `ArrayList.add()` gibi.

## Kapasiteyi önceden belirlemek

```java
StringBuilder report = new StringBuilder(5_000);
for (int i = 0; i < 1_000; i++) {
    report.append("satir-").append(i).append('\n');
}
```

Son uzunluk kabaca biliniyorsa, kurucuya ilk kapasiteyi vermek tekrarlanan kopyalama adımlarının tamamını ortadan kaldırır. `ensureCapacity(int)` da sonradan aynı amaçla çağrılabilir, ancak verilen değer mevcut kapasiteden küçükse hiçbir şey yapmaz; büyütme formülünü tetiklemez.

## Dikkat edilmesi gerekenler

- `length()` ile kapasite farklı şeylerdir; `length()` doldurulmuş karakter sayısını, kapasite ise iç dizinin boyutunu verir ve doğrudan sorgulanamaz, yalnızca `capacity()` ile okunabilir.
- `new StringBuilder(String str)` kurucusu, kapasiteyi `str.length() + 16` olarak ayarlar; bu yüzden bilinen bir başlangıç metniyle oluşturulan `StringBuilder`'lar ilk büyüme adımına diğerlerinden daha geç ulaşır.
- Çok büyük metinler için tekrarlı `+` ile `String` birleştirme her adımda yeni bir `String` nesnesi yarattığından, bu büyüme avantajından hiç yararlanamaz; derleyici bunu ancak basit, tek ifadelik birleştirmelerde örtük `StringBuilder`'a çevirir.

## Denemek için

```
jshell> var sb = new StringBuilder()
jshell> sb.capacity()
jshell> for (int i = 0; i < 17; i++) sb.append('a')
jshell> sb.capacity()
```

İlk `capacity()` çağrısı varsayılan `16` değerini gösterir; `17` karakter eklendiğinde kapasite dolup `34`'e büyür ve ikinci `capacity()` çağrısı bu artışı doğrular.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html]
