# charset-encoding-pitfalls

`String.getBytes()` ve `new String(byte[])` gibi yapıcı/metotların `Charset` parametresiz aşırı yüklemeleri, JVM'in `file.encoding` ile belirlenen platform varsayılan karakter kümesini kullanır. Bu varsayılan, programı derleyip test ettiğiniz makinede `UTF-8` olsa da üretim sunucusunda veya farklı bir işletim sisteminde başka bir değer olabilir; aynı kod, aynı girdiyle farklı makinelerde farklı bayt dizileri veya farklı metinler üretebilir. Charset'i her zaman açık belirtmek bu platforma bağımlılığı ortadan kaldırır.

## Varsayılan charset'in sessiz etkisi

```java
String metin = "merhaba dünya";

byte[] varsayilanBaytlar = metin.getBytes();
byte[] utf8Baytlar = metin.getBytes(StandardCharsets.UTF_8);

System.out.println(varsayilanBaytlar.length);
System.out.println(utf8Baytlar.length);
```

`ü` gibi ASCII dışı bir karakter `UTF-8`'de 2 bayt, `ISO-8859-1`'de 1 bayt ile kodlanır; `getBytes()` çağrısı hangi charset'i kullandığını parametre olarak söylemediği için bu uzunluk farkı, kodu okuyan birinin fark edemeyeceği bir platform bağımlılığıdır. Aynı belirsizlik `new String(bytes)` çağrısında da vardır: baytlar hangi charset ile üretildiyse metne geri dönüştürülürken de aynı charset verilmezse karakterler bozulur (`mojibake`).

## Dosya ve akış okuma/yazmada charset

```java
try (var writer = Files.newBufferedWriter(
        Path.of("notlar.txt"), StandardCharsets.UTF_8)) {
    writer.write("türkçe karakterler: ğ ş ı ö ü ç");
}

String icerik = Files.readString(Path.of("notlar.txt"), StandardCharsets.UTF_8);
```

`Files.newBufferedWriter` ve `Files.readString` gibi NIO metotları `Charset` parametresini zorunlu kılmaz ama kabul eder; parametre verilmediğinde dosya işlemleri de yine platform varsayılanına düşer. Yazarken ve okurken aynı charset'in açıkça belirtilmesi, dosyanın başka bir işletim sisteminde veya farklı `file.encoding` ayarıyla çalışan bir JVM'de okunduğunda da doğru sonuç vermesini garanti eder.

## Dikkat edilmesi gerekenler

- `Charset.defaultCharset()` ile çalışma anındaki varsayılanı sorgulayabilirsiniz, ama buna güvenmek yerine her zaman `StandardCharsets.UTF_8` gibi sabit bir değer geçirmek tercih edilmelidir.
- Java 18'den itibaren JVM'in platform varsayılan charset'i çoğu ortamda `UTF-8` olacak şekilde standardize edildi (JEP 400), ama bu eski davranışa dayanan kodu otomatik düzeltmez; eski sürümlerle uyumluluk gerekiyorsa charset'i açık belirtmek hâlâ zorunludur.
- `String(byte[], int, int)` gibi charset almayan yapıcılar da aynı varsayılan sorunu taşır; `Charset` veya charset adı (`String`) alan aşırı yüklemeler tercih edilmelidir.

## Denemek için

```
jshell> var b1 = "café".getBytes(java.nio.charset.StandardCharsets.UTF_8)
jshell> var b2 = "café".getBytes(java.nio.charset.StandardCharsets.ISO_8859_1)
jshell> b1.length
jshell> b2.length
```

`café` dizesinin `UTF-8` kodlamasında `é` karakteri 2 bayt tutarken `ISO-8859-1`'de aynı karakterin 1 bayt tuttuğunu, dolayısıyla iki dizinin toplam bayt uzunluklarının farklı çıktığını gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/charset/Charset.html]
