# Dizilerde toString() Tuzağı

Java'da diziler bir sınıf hiyerarşisine sahiptir ama kendi `toString()` ve `equals()` metotlarını tanımlamazlar; `Object`'ten miras aldıkları varsayılan davranışı kullanırlar. Bu yüzden `System.out.println(array)` gibi bir çağrı, elemanların içeriğini değil tipi ve hash kodunu gösteren anlamsız bir metin üretir. Yeni başlayanlar genellikle bunu bir bug zannedip saatlerce debug yapar.

## Neden hash kodu gibi bir şey basılıyor

```java
int[] numbers = {1, 2, 3};
System.out.println(numbers);           // [I@1540e19d
System.out.println(numbers.toString()); // aynı sonuç
```

`Object.toString()`, varsayılan olarak `getClass().getName() + "@" + Integer.toHexString(hashCode())` döner. Dizi sınıfları (`int[]` için `[I`, `String[]` için `[Ljava.lang.String;` gibi) bu metodu `override` etmez, dolayısıyla ekrana basılan şey dizinin içeriği değil, `Class` adının kısaltması ile nesnenin `identity hash code`'udur. Her çalıştırmada farklı bir değer görülmesi normaldir, çünkü bu hash bellekteki referansla ilişkilidir.

## Arrays.toString ve deepToString ile doğru basım

```java
int[] flat = {1, 2, 3};
int[][] nested = {{1, 2}, {3, 4}};

System.out.println(java.util.Arrays.toString(flat));      // [1, 2, 3]
System.out.println(java.util.Arrays.toString(nested));    // [[I@..., [I@...]  -- yanlis!
System.out.println(java.util.Arrays.deepToString(nested)); // [[1, 2], [3, 4]]
```

`Arrays.toString()` tek boyutlu dizileri doğru biçimde elemanlarına ayırarak yazdırır, ama çok boyutlu bir diziye uygulandığında iç dizileri yine kendi referans temsiliyle basar; çünkü `Arrays.toString(Object[])` her elemanı sadece kendi `toString()`'iyle çağırır ve iç eleman da bir dizi olduğu için aynı tuzağa düşer. Gerçekten iç içe geçmiş diziler için `Arrays.deepToString()` kullanılmalıdır, çünkü o her `Object[]` elemanını özyinelemeli (`recursive`) olarak kontrol eder.

## Dikkat edilmesi gerekenler

- `equals()` de aynı sorunu taşır: `new int[]{1,2}.equals(new int[]{1,2})` `false` döner çünkü karşılaştırma içerik bazlı değil referans bazlıdır; içerik karşılaştırması için `Arrays.equals()` veya iç içe diziler için `Arrays.deepEquals()` gerekir.
- `String.valueOf(array)` da `toString()`'i çağırdığı için aynı hash kodu problemini taşır.
- Bir dizi doğrudan `HashMap` anahtarı ya da `HashSet` elemanı yapılırsa, `hashCode()` da referans tabanlı kaldığından içeriği aynı iki dizi farklı anahtar sayılır; bu durumda `List` sarmalamak veya `Arrays.hashCode()` tabanlı bir sarmalayıcı kullanmak gerekir.

## Denemek için

```
jshell> int[] a = {1, 2, 3}
jshell> System.out.println(a)
jshell> System.out.println(java.util.Arrays.toString(a))
```

İlk `println` çağrısı `[I@...` gibi anlamsız bir referans temsili basar, ikincisi ise `[1, 2, 3]` yazdırarak diziyi içeriğiyle gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html]
