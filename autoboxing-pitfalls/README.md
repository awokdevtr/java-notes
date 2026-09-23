# Autoboxing Tuzakları

Java, ilkel tipler (`int`, `long`, `boolean`...) ile karşılık gelen sarmalayıcı sınıflar (`Integer`, `Long`, `Boolean`...) arasında dönüşümü derleyici seviyesinde otomatik yapar. Bu kolaylık genelde sorunsuz çalışır, ama sarmalayıcı tipin `null` olabileceği ya da beklenmedik bir noktada `unboxing` tetiklendiği durumlarda çalışma zamanında sessizce `NullPointerException` fırlatan ya da yanlış tipte değer üreten koda yol açar.

## null bir sarmalayıcıyı unboxing etmek

```java
Map<String, Integer> stok = new HashMap<>();
stok.put("elma", 5);

int adet = stok.get("armut"); // NullPointerException
```

`stok.get("armut")` anahtar bulunamadığı için `null` döner. Atama `int adet` ilkel tipe yapıldığından derleyici bunu arka planda `adet.intValue()` çağrısına çevirir; `null` referans üzerinde metot çağrısı yapıldığı için `NullPointerException` fırlar. Hata mesajı `intValue()` çağrısını gösterir, kaynak satırda görünür bir `null` kontrolü olmadığı için bu genelde ilk bakışta şaşırtıcıdır. Çözüm, ya sonucu `Integer` olarak tutup açıkça `null` kontrolü yapmak ya da `getOrDefault("armut", 0)` gibi varsayılan değerli bir API kullanmaktır.

## Ternary operatöründe gizli unboxing

```java
Integer bakiyeUyarisi = null;
boolean acil = true;

int sonuc = acil ? 1 : bakiyeUyarisi; // NullPointerException
```

`? :` operatöründe dallardan biri ilkel tip (`1`, yani `int`), diğeri sarmalayıcı tip (`bakiyeUyarisi`, `Integer`) olduğunda derleyici ifadenin ortak tipini ilkel `int` olarak belirler ve `Integer` dalını `intValue()` ile otomatik `unboxing` eder. Bu, çalışma zamanında hangi dal seçilirse seçilsin gerçekleşir; yani `acil` `true` olduğu ve `bakiyeUyarisi` hiç değerlendirilmeyecek gibi göründüğü halde, derleyicinin ürettiği bayt kodu `bakiyeUyarisi` üzerinde `unboxing` çağrısını içerdiğinden istisna yine de fırlar. Her iki dalın da sarmalayıcı tip olması (`(Integer) 1`) ya da `null` kontrolünün ternary dışına taşınması bu tuzağı önler.

## Dikkat edilmesi gerekenler

- `==` ile sarmalayıcı karşılaştırması ayrı bir tuzaktır (bkz. `integer-cache-pitfall`); burada asıl risk `null` üzerinde örtük `unboxing` çağrısıdır.
- Döngü içinde sık autoboxing/unboxing (`Long toplam = 0L; for (...) toplam += i;`) her adımda yeni bir sarmalayıcı nesne oluşturur; performansa duyarlı kodda ilkel tip biriktirici tercih edilmelidir.
- `Optional<Integer>` gibi API'lerde `.get()` sonucu doğrudan ilkel tipe atamadan önce `.isPresent()` ya da `.orElse()` ile kontrol edilmelidir.

## Denemek için

```
jshell> Map<String, Integer> m = new HashMap<>(); m.put("a", 1)
jshell> int x = m.get("b")
jshell> Integer y = null; int z = true ? 1 : y
```

İlk komut haritayı kurar, ikinci ve üçüncü komutlar `null` üzerinde örtük `unboxing` yüzünden `NullPointerException` fırlatır; hata çıktısındaki `intValue()` çağrısı derleyicinin eklediği dönüşümü gösterir.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html#jls-5.1.8]
