# var ile Yerel Değişken Tip Çıkarımı

Java 10 ile gelen `var`, bir yerel değişkenin tipini tekrar tekrar yazmak zorunda kalmadan derleyiciye çıkarttırmayı sağlar. Bu, JavaScript veya Python'daki dinamik tipleme ile karıştırılmamalı: değişken hâlâ derleme zamanında sabit, somut bir tipe bağlanır; `var` sadece o tipin kaynak kodda yazılmasını atlatan bir kısayoldur. Uzun generic tipler ya da açıkça anlaşılan sağ taraf ifadeleriyle çalışırken gürültüyü azaltır, ama yanlış kullanıldığında okunabilirliği de düşürebilir.

## Derleme zamanında somut tipe bağlanma

```java
var siparisler = new ArrayList<String>();
siparisler.add("kalem");

var toplam = siparisler.size() * 2; // int
```

Burada `siparisler` derleyici tarafından `ArrayList<String>` olarak çıkarılır, `Object` veya başka bir üst tip değil; bu yüzden `siparisler.add(42)` derleme hatası verir. `var`, sağ taraftaki ifadenin statik tipini alıp değişkene atar ve bu tip daha sonra değişmez; `siparisler` başka bir noktada farklı bir tipteki koleksiyona yeniden atanamaz. IDE'lerin ve `javac`'ın gösterdiği hata mesajları çıkarılan gerçek tipi (`ArrayList<String>`) referans alır, `var` kelimesini değil.

## Tip çıkarımının imkansız olduğu durumlar

```java
var x = null;              // derleme hatası: tip çıkarılamıyor
var dizi = { 1, 2, 3 };    // derleme hatası: array initializer

var gecerliDizi = new int[] { 1, 2, 3 }; // olur
```

`var`, sağ taraftaki ifadenin tipine bakarak çalıştığı için sağ tarafın kendisi tip bilgisi taşımayan durumlarda çıkarım yapılamaz. `null` literalinin tek başına bir tipi yoktur, dolayısıyla derleyici `x`'e hangi tipi vereceğini bilemez. Aynı şekilde süslü parantezli array initializer sözdizimi (`{1, 2, 3}`) yalnızca açık bir dizi tipi bildirildiğinde (`int[] dizi = {1, 2, 3};`) geçerlidir; `var` ile birlikte kullanılamaz çünkü derleyici initializer'ı hangi dizi tipine dönüştüreceğini çıkaramaz. `new int[]{...}` gibi tipi açıkça taşıyan bir ifadeyle bu sorun ortadan kalkar.

## Dikkat edilmesi gerekenler

- `var` yalnızca yerel değişkenler (metot içi, for döngü sayaçları, try-with-resources kaynakları) için kullanılabilir; alan (field), metot parametresi ve dönüş tipi bildiriminde geçersizdir.
- Sağ taraf açık değilse (`var sonuc = hesapla();` gibi metot adından tip anlaşılmıyorsa) okunabilirlik düşer; tip isimden veya bağlamdan net çıkmıyorsa açık tip yazmak tercih edilmelidir.
- `var` ile bildirilen değişken hâlâ statik tiplidir: derleyici hatalarını ve IDE tamamlamasını etkilemez, sadece kaynak kodda tip adının tekrarını kaldırır.

## Denemek için

```
jshell> var liste = new ArrayList<String>()
jshell> liste.add("a")
jshell> /var liste
```

İlk komut `liste`'nin tipini çıkarır, `/var` komutu jshell'de değişkenin gerçek statik tipinin `ArrayList<String>` olarak kaydedildiğini gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/language/local-variable-type-inference.html]
