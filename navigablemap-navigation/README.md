# navigablemap-navigation

`TreeMap`'te belirli bir anahtara en yakın girdiyi bulmak gerektiğinde, tüm anahtarları gezip elle karşılaştırma yapmak hem okunaksız hem de `O(n)` maliyetlidir. `NavigableMap` arayüzü, sıralı anahtar kümesi üzerinde "bundan küçük/eşit en yakın", "bundan büyük/eşit en yakın" gibi sorguları `O(log n)`'de yanıtlayan `floor`, `ceiling`, `higher` ve `lower` metotlarını sunar.

## floor / ceiling: eşitliği dahil eden arama

```java
NavigableMap<Integer, String> map = new TreeMap<>();
map.put(10, "on");
map.put(20, "yirmi");
map.put(30, "otuz");

System.out.println(map.floorKey(20));   // 20  (<=20 en büyük anahtar)
System.out.println(map.floorKey(25));   // 20  (<=25 en büyük anahtar)
System.out.println(map.ceilingKey(20)); // 20  (>=20 en küçük anahtar)
System.out.println(map.ceilingKey(21)); // 30  (>=21 en küçük anahtar)
```

`floorKey` aranan değere eşit veya ondan küçük en büyük anahtarı, `ceilingKey` ise eşit veya ondan büyük en küçük anahtarı döndürür; her ikisi de arama anahtarının kendisini eşleşme olarak kabul eder. Karşılık gelen bir anahtar yoksa (örneğin en küçük anahtardan daha düşük bir `floorKey` araması) `null` döner, bu yüzden sonucu kullanmadan önce `null` kontrolü gerekir.

## higher / lower: eşitliği dışlayan arama

```java
System.out.println(map.lowerKey(20));  // 10  (<20, eşiti saymaz)
System.out.println(map.higherKey(20)); // 30  (>20, eşiti saymaz)
```

`lowerKey` ve `higherKey`, `floorKey`/`ceilingKey` ile aynı yönde ararken aranan değerin kendisini eşleşme olarak kabul etmez; bu fark özellikle kuyruk/sıradaki eleman mantığı kurarken (bir anahtarın "kesin" öncülünü veya ardılını bulmak) önemlidir. Dört metodun da `Entry` döndüren karşılıkları (`floorEntry`, `ceilingEntry` vb.) vardır ve bunlar anahtarla birlikte değere de tek sorguda erişim sağlar.

## Dikkat edilmesi gerekenler

- Bu metotlar `HashMap`'te yoktur; sıralı gezinme gerektiren her yerde `TreeMap` veya başka bir `NavigableMap` implementasyonu gerekir.
- Her çağrı anahtar kümesinde `O(log n)` ikili arama yapar; döngü içinde tekrar tekrar çağırmak yerine gerekiyorsa `headMap`/`tailMap` ile alt küme üzerinde iterasyon tercih edilmelidir.
- Dönüş değeri `null` olabileceğinden, özellikle `Integer` gibi kutulanmış tiplerde doğrudan ilkel tipe atamak `NullPointerException`'a yol açabilir.

## Denemek için

```
jshell> var map = new java.util.TreeMap<Integer,String>()
jshell> map.put(10,"on"); map.put(20,"yirmi"); map.put(30,"otuz")
jshell> map.floorKey(25)
jshell> map.ceilingEntry(21)
```

`floorKey(25)` değeri `20` döner; `ceilingEntry(21)` ise `30=otuz` girdisini verir, çünkü 21'den büyük veya eşit en küçük anahtar 30'dur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/NavigableMap.html]
