# String.split() Tuzakları

`"a,b,,c,".split(",")` çağrısının kaç elemanlı bir dizi döndürdüğünü tahmin edin. Çoğu kişi 5 der, doğru cevap 4'tür. `String.split()` kullanışlı bir kısayol gibi görünür ama hem argümanını regex olarak yorumlar hem de sondaki boş parçaları varsayılan olarak siler; bu iki davranış birleştiğinde CSV benzeri veriyi ayrıştırırken sessiz veri kaybına yol açabilir.

## Sondaki boş string'lerin düşmesi

```java
String[] parts = "a,b,,c,".split(",");
System.out.println(parts.length);
System.out.println(Arrays.toString(parts));
```

Bu kod `4` yazdırır: `[a, b, , c]`. Dizedeki son `,` karakterinden sonraki boş parça otomatik olarak atılır; ortadaki boş parça (`b` ile `c` arasındaki) ise korunur. `split(String)`, iki parametreli `split(String, int)` metodunu `limit` değeri `0` ile çağırır ve `limit == 0` olduğunda sondaki boş eşleşmeler sonuçtan çıkarılır. Sabit sütun sayısı bekleyen bir CSV ayrıştırıcısı bu yüzden son sütunları boşsa satırı yanlış sayıda alana böler.

## limit parametresiyle kontrol

```java
String[] keepAll = "a,b,,c,".split(",", -1);
String[] firstTwo = "a,b,,c,".split(",", 2);

System.out.println(Arrays.toString(keepAll));
System.out.println(Arrays.toString(firstTwo));
```

Negatif bir `limit` verildiğinde sondaki boş parçalar da korunur ve `[a, b, , c, ]` elde edilir. Pozitif bir `limit` ise dizinin en fazla kaç eleman içereceğini sınırlar; son eleman, o noktaya kadar bölünmemiş kalan metni taşır. Yani `split(regex, -1)` "hiçbir şeyi atma" anlamına gelirken `split(regex)` ile aynı şey değildir.

## Dikkat edilmesi gerekenler

- `split()`'in ilk argümanı düz metin değil regex'tir; `"."` veya `"|"` gibi özel karakterleri ayraç olarak kullanmak için `Pattern.quote()` ile kaçış gerekir.
- Elemanların tamamını, boşlar dahil korumak istiyorsanız `limit` değerini `-1` verin.
- Java 8+'ta sabit bir ayraçla basit bölme için `Pattern.compile(regex).splitAsStream(input)` veya tek seferlik kullanımda doğrudan `split()` tercih edilebilir; ama döngü içinde tekrar tekrar aynı regex'i bölüyorsanız `Pattern`'i önceden derleyip yeniden kullanmak daha verimlidir.

## Denemek için

```
jshell> "a,b,,c,".split(",")
jshell> "a,b,,c,".split(",", -1)
jshell> "a,b,,c,".split(",", 2)
```

İlk çağrının son boş parçayı attığını, ikincisinin tüm parçaları koruduğunu, üçüncüsünün ise sonucu iki elemanla sınırlandırdığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#split(java.lang.String,int)]
