# Arrays.equals vs Arrays.deepEquals Tuzağı

İki diziyi karşılaştırırken `Arrays.equals(a, b)` kullanmak doğal bir refleks gibi görünür ve tek boyutlu, primitif elemanlı dizilerde beklenen sonucu verir. Ancak dizi içinde dizi (nested array, örneğin `int[][]` veya `Object[]` içinde başka bir `Object[]`) varsa `Arrays.equals` her elemanı yalnızca `equals()` ile karşılaştırır; iç dizilerin kendisi `equals()`'ı override etmediğinden bu karşılaştırma referans eşitliğine düşer ve içerikleri birbirinin aynısı olan iki dizi "farklı" çıkar.

## Yanıltıcı sonuç

```java
int[][] a = { {1, 2}, {3, 4} };
int[][] b = { {1, 2}, {3, 4} };

System.out.println(Arrays.equals(a, b)); // false
```

`Arrays.equals(Object[], Object[])` her bir `a[i]` ile `b[i]`'yi `equals()` ile karşılaştırır; burada elemanların kendisi `int[]` olduğundan ve diziler `Object`'ten gelen varsayılan `equals()`'ı kullandığından karşılaştırma iki ayrı nesnenin bellek referansını kıyaslar. İçerikleri birebir aynı olsa bile `a[0]` ile `b[0]` ayrı nesnelerdir, bu yüzden sonuç `false` olur.

## Doğru karşılaştırma

```java
System.out.println(Arrays.deepEquals(a, b)); // true

String icerik = Arrays.deepToString(a); // [[1, 2], [3, 4]]
```

`Arrays.deepEquals` her eleman çifti için, eğer eleman kendisi bir dizi ise özyinelemeli (recursive) olarak tekrar `deepEquals` çağırır, değilse normal `equals()`'a düşer. Bu sayede iç içe geçmiş diziler içerik bazlı karşılaştırılır. `Arrays.deepToString` da aynı mantıkla çalışır ve hata ayıklarken (`debug`) iç içe dizileri `toString()`'in verdiği anlamsız referans metnine (`[I@1b6d3586`) düşmeden okunabilir basmak için kullanılır.

## Dikkat edilmesi gerekenler

- `Arrays.equals` tek boyutlu primitif dizilerde (`int[]`, `double[]` vb.) doğru çalışır çünkü o aşırı yüklenmiş (`overload`) sürüm değerleri doğrudan karşılaştırır; sorun yalnızca dizi-içinde-dizi veya `Object[]` içinde karmaşık nesneler olduğunda ortaya çıkar.
- `HashMap` anahtarı ya da `Set` elemanı olarak nested array kullanmak aynı köke dayanan bir tuzaktır: dizilerin `hashCode()`'u da içeriğe göre değil referansa göre hesaplanır, bu yüzden içerik aynı olsa bile farklı `hashCode` üretirler.
- `==` operatörü zaten her zaman referans karşılaştırır; asıl tuzak `equals()`'ın "içerik karşılaştırır" beklentisiyle çağrılıp diziler için bu beklentiyi karşılamamasıdır.

## Denemek için

```
jshell> int[][] a = { {1, 2}, {3, 4} }
jshell> int[][] b = { {1, 2}, {3, 4} }
jshell> java.util.Arrays.equals(a, b)
jshell> java.util.Arrays.deepEquals(a, b)
```

İlk karşılaştırma `false`, ikincisi `true` döner; aynı içerikteki iki dizi için verilen bu iki farklı cevap, `equals` ile `deepEquals` arasındaki farkın neden göz ardı edilemeyecek kadar önemli olduğunu gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#deepEquals(java.lang.Object%5B%5D,java.lang.Object%5B%5D)]
