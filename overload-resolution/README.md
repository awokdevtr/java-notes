# Overload Çözümlemesi

Aynı isimde birden fazla metot tanımlandığında (`overload`), derleyici hangisinin çağrılacağına tek bir adımda değil, üç aşamalı bir algoritmayla karar verir. Bu sıralamayı bilmemek, `autoboxing` ve `varargs` bir arada kullanıldığında hangi metodun çalışacağını tahmin etmeyi zorlaştırır ve çalışma zamanında değil ama "yanlış overload çağrıldı" şeklinde sessiz mantık hatalarına yol açar.

## Üç aşamalı seçim

```java
static void yaz(int x)          { System.out.println("int: " + x); }
static void yaz(long x)         { System.out.println("long: " + x); }
static void yaz(Integer x)      { System.out.println("Integer: " + x); }
static void yaz(int... x)       { System.out.println("varargs: " + x.length); }

yaz(5); // "int: 5" basılır
```

Derleyici önce **faz 1**'i dener: `autoboxing`/`unboxing` ve `varargs` olmadan, yalnızca ilkel genişletme (`int` → `long` gibi) ile tam eşleşen bir `overload` arar. `yaz(5)` çağrısında `yaz(int)` bu fazda doğrudan eşleştiği için seçilir; `yaz(long)`, `yaz(Integer)` ve `yaz(int...)` hiç değerlendirilmez. Faz 1'de eşleşme bulunamazsa derleyici **faz 2**'ye geçer ve `autoboxing`/`unboxing`'e izin verir; orada da eşleşme yoksa son çare olarak **faz 3**'te `varargs` metotları dener.

## Autoboxing ve varargs birlikte

```java
static void oyna(Integer x) { System.out.println("Integer overload"); }
static void oyna(int... x)  { System.out.println("varargs overload"); }

oyna(5); // "Integer overload" basılır
```

Burada `int` değeri hem `Integer`'a kutulanarak (faz 2) hem de tek elemanlı bir `int[]`'e sarılarak (faz 3) `oyna` metoduna uyabilir; ama derleyici fazları sırayla dener ve faz 2'de zaten bir eşleşme bulduğu için faz 3'e hiç bakmaz. Sonuç olarak `Integer` alan `overload` kazanır. Bu davranış, `varargs` içeren bir API'ye sonradan tek bir kutulanmış parametre alan `overload` eklemenin, mevcut çağrıların hedefini sessizce değiştirebileceği anlamına gelir.

## Dikkat edilmesi gerekenler

- `null` geçildiğinde en spesifik referans tipini alan `overload` seçilir; birden fazla eşit derecede spesifik aday varsa derleme zamanı hatası (`ambiguous method call`) alınır.
- Faz sırası her zaman sabittir: önce kutulama yok, sonra kutulama var, en son `varargs`. Bu sıralamayı ezbere bilmek, hangi `overload`'un neden seçildiğini debug etmeden anlamayı sağlar.
- Aynı isimde hem sabit parametreli hem `varargs` `overload` tanımlamak, API'ye yeni bir sabit parametreli `overload` eklendiğinde eski çağrıların hedefinin değişmesine yol açabileceğinden genelde kaçınılması gereken bir tasarımdır.

## Denemek için

```
jshell> void f(long x) { System.out.println("long"); }
jshell> void f(Integer x) { System.out.println("Integer"); }
jshell> void f(int... x) { System.out.println("varargs"); }
jshell> f(5)
```

`int` değeri hem `long`'a genişletilerek (faz 1) hem kutulanarak (faz 2) hem de `varargs`'a sarılarak (faz 3) eşleşebilir; çıktı olarak "long" göreceksiniz çünkü faz 1'de bulunan eşleşme diğer fazlara hiç bakılmadan kazanır.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.2]
