# List.of() ve Collections.unmodifiableList() Farkı

"Değiştirilemez liste" lazım olduğunda çoğu kişi `Collections.unmodifiableList()` yazar ve işin bittiğini düşünür. Oysa bu metot gerçek bir değişmezlik sağlamaz; sadece orijinal listenin üzerine ince bir "sadece okunabilir" kaplama koyar. Arkadaki liste değişirse kaplama da değişir, çünkü aslında iki farklı referans aynı veriyi gösteriyordur.

## unmodifiableList: bir görünümden ibaret

```java
List<String> kaynak = new ArrayList<>(List.of("a", "b"));
List<String> gorunum = Collections.unmodifiableList(kaynak);

kaynak.add("c");
System.out.println(gorunum); // [a, b, c]

gorunum.add("d"); // UnsupportedOperationException
```

`gorunum` kendi başına `add` çağrısını reddeder, ama bu sahte bir güvenliktir: `kaynak` referansına hâlâ erişimi olan herhangi bir kod listeyi değiştirdiğinde, `gorunum` üzerinden okuyan her yer bu değişikliği anında görür. `unmodifiableList` bu yüzden "salt okunur görünüm" olarak adlandırılır, "değişmez liste" değil.

## List.of(): gerçek ve bağımsız bir kopya

```java
List<String> sabit = List.of("a", "b", "c");

sabit.add("d");          // UnsupportedOperationException
List.of("a", null, "c"); // NullPointerException
```

`List.of(...)` verilen elemanları kendi iç yapısına alır ve dışarıdaki hiçbir referansa bağımlı olmayan, tamamen bağımsız bir liste üretir; kaynak değişse de bu listeyi etkilemez, çünkü zaten kaynağa bir bağı yoktur. Ayrıca `null` elemana izin vermez ve bunu oluşturma anında, geç fark edilecek bir hataya dönüşmeden önce fırlatır. Mevcut bir koleksiyondan aynı garantiyle bağımsız kopya çıkarmak için `List.copyOf(koleksiyon)` kullanılır.

## Dikkat edilmesi gerekenler

- `unmodifiableList` yalnızca döndürdüğü referansı korur; arkadaki listeye erişimi olan kod hâlâ veriyi değiştirebilir.
- `List.of()`/`List.copyOf()` elemanları kopyaladığı için bu sızıntı riski yoktur, ama bu kopyalama maliyeti büyük koleksiyonlarda göz önünde bulundurulmalı.
- İkisi de değiştirme girişiminde aynı istisnayı (`UnsupportedOperationException`) fırlatır; farkı anlamak için istisnanın ne zaman geldiğine değil, verinin kimin elinde olduğuna bakmak gerekir.

## Denemek için

```
jshell> var kaynak = new java.util.ArrayList<>(java.util.List.of("x", "y"))
jshell> var gorunum = java.util.Collections.unmodifiableList(kaynak)
jshell> kaynak.add("z")
jshell> gorunum
```

Son satır `[x, y, z]` döndürür; `gorunum` hiçbir değişiklik yapmamış olsa da arkadaki `kaynak` listesinin büyümesini yansıtır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#of(E...)]
