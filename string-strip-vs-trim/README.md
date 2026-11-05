# string-strip-vs-trim

`String.trim()` yıllardır boşluk temizlemek için kullanılan standart metottur, fakat "boşluk" tanımı Unicode'un tanıdığı boşluk karakterlerinin yalnızca bir kısmını kapsar. Java 11 ile gelen `strip()` ailesi, bu tanımı `Character.isWhitespace` ile Unicode'a uyumlu hale getirerek `trim()`'in gözden kaçırdığı karakterleri de temizler.

## trim() neyi boşluk sayar

```java
String s = " merhaba "; // no-break space (U+00A0)
System.out.println(s.trim().equals(" merhaba ")); // true: hic kirpilmedi
System.out.println(s.strip().equals("merhaba"));            // true: kirpildi
```

`trim()`, kod noktası `' '` (ASCII boşluk) veya küçük eşit olan her karakteri boşluk sayar ve bu aralığı sabit bir eşik değeriyle kontrol eder. ` ` (no-break space) gibi Latin-1 kaynaklı ya da diğer Unicode boşluk karakterleri bu eşiğin üzerinde kod noktalarına sahiptir, dolayısıyla `trim()` tarafından görmezden gelinir. `strip()` ise her karakteri `Character.isWhitespace()` ile sınar, bu metot Unicode'un resmi boşluk kategorisini tanır ve ` `'ı da kırpar.

## stripLeading() ve stripTrailing()

```java
String padded = "  log satiri  ";
System.out.println("[" + padded.stripLeading() + "]");  // "[log satiri  ]"
System.out.println("[" + padded.stripTrailing() + "]"); // "[  log satiri]"
```

`trim()`'in tek yönlü bir karşılığı yoktur; baştan veya sondan kırpmak için manuel `indexOf`/`substring` yazmak gerekirdi. `stripLeading()` ve `stripTrailing()` bu iki yönü ayrı ayrı sunar, ikisi de `strip()` ile aynı Unicode farkındalığına sahiptir.

## Dikkat edilmesi gerekenler

- `isBlank()` de aynı Unicode kurallarını kullanır: bir string yalnızca `strip()` ile temizlenebilecek karakterlerden oluşuyorsa `true` döner.
- Performans farkı pratikte önemsizdir; asıl fark doğruluktur, özellikle dış kaynaklardan (web formları, dosyalar, veritabanları) gelen metinlerde no-break space gibi görünmez karakterler sık rastlanır.
- `strip()` de `trim()` gibi yeni bir `String` döndürür; orijinal değişmez.

## Denemek için

```
jshell> " test ".trim().length()
jshell> " test ".strip().length()
```

İlk satır `6`, ikinci satır `4` döner; `trim()`'in no-break space'leri saymadığını, `strip()`'in ise onları kırptığını doğrudan görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#strip()]
