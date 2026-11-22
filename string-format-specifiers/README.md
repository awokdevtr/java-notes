# String.format ve Formatter Dönüşüm Karakterleri

Bir sayıyı iki ondalık basamakla, bir tarihi belirli bir genişlikte veya bir tabloyu hizalı sütunlarla yazdırmak isteyince çoğu kişi elle `substring`/`padding` mantığı kurmaya kalkar. `String.format` (ve altındaki `java.util.Formatter`), `printf`-tarzı dönüşüm karakterleriyle bu biçimlendirmeyi tek satırda, yerelleştirmeye de açık biçimde yapar.

## Temel dönüşüm karakterleri

```java
String s1 = String.format("%d adet, toplam %.2f TL", 3, 19.9);
// "3 adet, toplam 19.90 TL"

String s2 = String.format("[%5d]", 42);   // "[   42]"  -> sağa hizalı, genişlik 5
String s3 = String.format("[%-5d]", 42);  // "[42   ]"  -> solda hizalı
String s4 = String.format("[%05d]", 42);  // "[00042]"  -> sıfır dolgulu
```

`%d` tamsayı, `%f` ondalıklı sayı, `%s` herhangi bir nesnenin `toString()`'ini, `%c` tek karakteri temsil eder. Dönüşüm karakterinden önceki genişlik (`5`), bayraklar (`-` sol hizalama, `0` sıfır dolgu) ve `.` ile başlayan kesinlik (`%.2f`'teki `2` ondalık basamak) hepsi aynı `%[bayrak][genişlik][.kesinlik]dönüşüm` kalıbına uyar.

## Yerel ayara (Locale) duyarlılık ve argüman indeksleme

```java
double tutar = 1234.5;
System.out.println(String.format(Locale.US, "%,.2f", tutar));   // 1,234.50
System.out.println(String.format(Locale.of("tr", "TR"), "%,.2f", tutar)); // 1.234,50

// Aynı argümanı birden fazla yerde kullanma
String.format("%1$s, tekrar: %1$s", "merhaba"); // "merhaba, tekrar: merhaba"
```

`,` bayrağı binlik ayırıcıyı ekler ama ayırıcı ve ondalık karakterin kendisi verilen `Locale`'e göre değişir; `Locale` parametresi verilmezse `String.format`, JVM'in varsayılan `Locale`'ini kullanır ve bu da `locale-case-conversion` konusundakine benzer şekilde makineden makineye farklı çıktıya yol açabilir. `N$` sözdizimi ise argümanı pozisyonuna göre adlandırarak aynı değeri format dizesinde birden çok yerde tekrar kullanmanıza izin verir.

## Dikkat edilmesi gerekenler

- `%d` sadece tamsayı türleriyle (`int`, `long`, `Integer`...) çalışır; bir `double`'a `%d` vermek `IllegalFormatConversionException` fırlatır, ondalıklı sayılar için `%f`/`%e`/`%g` gerekir.
- Format dizesindeki argüman sayısı ile verilen argüman sayısı uyuşmazsa `MissingFormatArgumentException` çalışma zamanında fırlatılır; derleyici bunu yakalamaz.
- Raporlama/log gibi makine tarafında okunacak çıktılarda `Locale`'i açıkça belirtmek (örn. `Locale.ROOT`), kullanıcıya gösterilecek metinlerde ise kullanıcının `Locale`'ini kullanmak gerekir — ikisini karıştırmak sayı ayırıcılarını bozar.

## Denemek için

```
jshell> String.format("[%8.2f]", 3.14159)
jshell> String.format(Locale.of("tr","TR"), "%,d", 1000000)
jshell> String.format("%2$s once %1$s sonra", "once", "simdi")
```

İlk satırda genişlik ve kesinlik birlikte kullanıldığında sayının nasıl hem kırpıldığını hem de hizalandığını, ikinci satırda Türkçe locale'de binlik ayırıcının noktaya döndüğünü görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Formatter.html]
