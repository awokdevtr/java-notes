# switch icinde case null

Klasik `switch`, secici (`selector`) ifadesi `null` oldugunda her zaman `NullPointerException` firlatir; bu kurali hem eski `case X:` sozdizimi hem de yeni `switch expression` miras alir. Kod bu yuzden `switch`'ten once ayri bir `null` kontrolu yazmak zorunda kalirdi. Java 21, `case null ->` etiketiyle bu kontrolu doğrudan `switch`'in icine tasir.

## default, null'u otomatik kapsamaz

```java
String renk = null;
String sonuc = switch (renk) {
    case "kirmizi" -> "sicak";
    case "mavi"    -> "serin";
    default        -> "bilinmeyen";
};
```

Bu kod calisirken `NullPointerException` firlatir, cunku `default` dali yalnizca eslesen baska bir `case` olmadigi durumlari kapsar, `null` secicisini kapsamaz. `null`, hicbir sabit degere veya `pattern`'e esit sayilmadigi icin ayri bir eslesme kurali gerektirir.

## case null ile acik kontrol

```java
String sonuc2 = switch (renk) {
    case null       -> "renk yok";
    case "kirmizi"  -> "sicak";
    case "mavi"     -> "serin";
    default         -> "bilinmeyen";
};
```

`case null ->` dali eklenince `null` secici artik istisna firlatmadan bu dala duser. `null` ve `default` davranisi ayni olan durumlarda `case null, default ->` seklinde tek satirda birlestirilebilir; bu, ayri bir `if (x == null)` blogu yazmaktan daha az kod ve daha az unutma riski demektir.

## Dikkat edilmesi gerekenler

- `case null` yalnizca `switch expression` ve pattern destekli `switch`'lerde gecerlidir; `case null` yazmazsaniz eski davranis (NPE) devam eder.
- `case null` bir kez yazilabilir ve genellikle en basa konur; sabit deger veya `pattern` dallarindan once ya da sonra olmasi sonucu degistirmez, sira sadece okunabilirlik icin onemlidir.
- `case null, default ->` kisayolu, `null` ile "digerleri" icin ayni sonuc uretilecekse tercih edilir; farkli sonuc gerekiyorsa ikisi ayri dallar olarak yazilmalidir.

## Denemek icin

```
jshell> String renk = null
jshell> String s = switch (renk) { case "kirmizi" -> "sicak"; default -> "bilinmeyen"; }
jshell> String s2 = switch (renk) { case null -> "renk yok"; default -> "bilinmeyen"; }
```

Ikinci satir `NullPointerException` firlatirken ucuncu satir `"renk yok"` degerini doner.

[Kaynak: https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch-statements-and-expressions.html]
