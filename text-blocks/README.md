# text-blocks

Çok satırlı bir JSON, SQL sorgusu ya da HTML parçasını klasik `String` literaliyle yazmak, her satır sonuna `\n` eklemeyi ve tırnak işaretlerini `\"` ile kaçırmayı gerektirir; sonuç hem okunması zor hem de düzenlemesi hataya açık bir kod olur. Java 15 ile kalıcı hale gelen `text block` (`"""` ile sınırlanan literal), çok satırlı metinleri kaynak koddaki görünümüyle neredeyse birebir korur.

## Temel kullanım

```java
String json = """
    {
        "name": "Ada",
        "role": "engineer"
    }
    """;
```

Açılış `"""` işaretinden hemen sonra satır sonu gelmelidir; içerik bir sonraki satırdan başlar. Derleyici, tüm satırlardaki ortak baştaki boşluğu otomatik olarak keser (`incidental white space` temizliği), bu yüzden kaynak kodda okunabilirlik için girinti kullanabilirsiniz ama üretilen `String`'de bu fazladan boşluk yer almaz. Kapanış `"""` işaretinin sütun konumu, kesilecek boşluk miktarını belirler.

## Kaçış karakterleri ve satır sonu kontrolü

```java
String row = """
    Ad: Ada\tSoyad: Lovelace \
    (aynı satırda devam eder)""";
```

Text block içinde çift tırnak `"` genellikle kaçırılmadan yazılabilir; sadece üçlü tırnak dizisiyle çakışma riski varsa `\"` gerekir. Satır sonunda ters eğik çizgi (`\`) koymak, o satırın sonundaki yeni satır karakterini bastırır ve mantıksal satırı bir sonrakiyle birleştirir. `\s` kaçışı ise satır sonunda otomatik kırpılan boşlukları korumak için kullanılır.

## Ne zaman işe yarar

- Gömülü SQL, JSON, HTML veya regex gibi çok satırlı, alıntı işareti yoğun içerikler.
- Test kodunda beklenen çıktı metinlerini okunabilir tutmak.
- Tek satırlık kısa string'ler için gereksizdir; klasik literal daha nettir.

## Denemek için

```
jshell> String s = """
   ...>     merhaba
   ...>     dünya
   ...>     """;
jshell> System.out.println(s.lines().count());
```

`lines().count()` çağrısının `2` döndürdüğünü, baştaki ortak girintinin `String` içeriğine dahil edilmediğini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/17/text-blocks/index.html]
