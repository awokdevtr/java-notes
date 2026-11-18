# unnamed variables (_)

Bir değişkeni sadece dil kuralını yerine getirmek için tanımlayıp hiç kullanmadığınız durumlar yaygındır: bir `catch` bloğunda exception nesnesiyle ilgilenmemek, bir lambda'nın parametresini kullanmamak ya da bir `switch` pattern'inde bazı bileşenleri önemsememek. Bu durumlarda anlamsız bir isim (`e`, `ignored`, `unused`) uydurmak gerekir ve derleyici "kullanılmayan değişken" uyarısı verir. Java 22 ile gelen `_` (unnamed variable), bu değişkenlere isim vermeyi tamamen gereksiz kılar.

## catch ve lambda'da kullanım

```java
try {
    Integer.parseInt(input);
} catch (NumberFormatException _) {
    System.out.println("gecersiz sayi");
}

map.forEach((_, value) -> System.out.println(value));
```

`catch (NumberFormatException _)` yazıldığında exception nesnesine hiç erişilemez; derleyici bunu bilinçli bir "önemsemiyorum" bildirimi olarak görür ve kullanılmayan değişken uyarısı üretmez. `_`, normal bir tanımlayıcı değildir — aynı `scope` içinde birden fazla kez kullanılabilir, çünkü her biri birbirinden bağımsız, isimsiz bir yer tutucudur.

## record pattern'de bileşen eleme

```java
record Point(int x, int y) {}

if (obj instanceof Point(var x, var _)) {
    System.out.println("x koordinati: " + x);
}
```

`record pattern` ile `destructuring` yaparken sadece `x`'le ilgileniyorsanız `y` için `_` yazmak, o bileşenin kasıtlı olarak göz ardı edildiğini açıkça belirtir. Bu, `var y` yazıp sonra hiç kullanmamaktan farklıdır: `_` derleyiciye ve okuyana aynı anda "bu değer önemli değil" der.

## Dikkat edilmesi gerekenler

- `_` sadece tek başına bir `declaration` olarak kullanılabilir; `_.someMethod()` gibi bir ifadede değişken olarak okunamaz.
- Geleneksel olarak `_` adında bir değişken veya metot tanımlayan eski kod, Java 22'den itibaren derleme zamanında uyarı alır ve gelecekte tamamen yasaklanması planlanır.
- Normal, kullanılacak bir değişken için `_` kullanmayın; bu yalnızca "bu değere gerçekten ihtiyacım yok" durumları içindir.

## Denemek için

```
jshell> record Pair(int a, int b) {}
jshell> Object o = new Pair(1, 2)
jshell> if (o instanceof Pair(var _, var b)) System.out.println(b)
```

`a` bileşeni `_` ile göz ardı edilir, sadece `b` yazdırılır — derleyici `a` için "kullanılmadı" uyarısı vermez çünkü zaten isimsizdir.

[Kaynak: https://docs.oracle.com/en/java/javase/22/language/unnamed-variables-and-patterns.html]
