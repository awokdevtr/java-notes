# EnumMap ve EnumSet

Bir enum sabitine göre anahtarlanan veri tutmak için `HashMap<Gun, Plan>` ya da `HashSet<Gun>` kullanmak işe yarar ama gereğinden pahalıdır: her anahtar için `hashCode()` hesaplanır, çarpışma zinciri aranır, kova (bucket) dizisi ayrılır. Enum sabitlerinin sayısı sabit ve her birinin bir `ordinal()` değeri olduğu için, JDK bu durumu dizi tabanlı özel koleksiyonlarla çözer: `EnumMap` ve `EnumSet`.

## Ordinal tabanlı dizi olarak EnumMap

```java
enum Gun { PAZARTESI, SALI, CARSAMBA, PERSEMBE, CUMA, CUMARTESI, PAZAR }

Map<Gun, String> plan = new EnumMap<>(Gun.class);
plan.put(Gun.CUMA, "deploy");
plan.put(Gun.PAZARTESI, "planlama");

for (var entry : plan.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}
```

`EnumMap` içeride `ordinal()` değeriyle indekslenen düz bir dizi tutar; `get`/`put` doğrudan dizi erişimine döner ve `hashCode()`/`equals()` hiç çağrılmaz. İterasyon sırası da `ordinal()` sırasına göredir, yani yukarıdaki döngü eklenme sırasından bağımsız olarak `PAZARTESI` önce `CUMA` sonra basılır. Kurucuya enum sınıfının `Class` nesnesinin verilmesi şarttır çünkü dizinin boyutu o enum'un sabit sayısından belirlenir.

## Bit vektörü olarak EnumSet

```java
enum Izin { OKU, YAZ, CALISTIR }

EnumSet<Izin> okuYaz = EnumSet.of(Izin.OKU, Izin.YAZ);
EnumSet<Izin> hepsi = EnumSet.allOf(Izin.class);
EnumSet<Izin> tersi = EnumSet.complementOf(okuYaz);
```

`EnumSet`, her sabiti kümedeki varlığına göre tek bir bit'e karşılık getiren bir bit vektörüyle temsil edilir; 64 veya daha az sabitli enum'larda bu tek bir `long` alanına sığar. `add`, `contains`, `remove` ve küme işlemleri (`complementOf`, birleşim, kesişim) bu haliyle bitwise operasyonlara indiği için `HashSet`'ten belirgin biçimde hızlıdır ve `null` eleman kabul etmez.

## Ne zaman işe yarar

- Anahtar veya eleman kümesi tamamen bir enum'un sabitlerinden oluşuyorsa, `HashMap`/`HashSet` yerine `EnumMap`/`EnumSet` tercih edilmelidir; performans kazancı yanında `ordinal()` sırasına göre öngörülebilir iterasyon da bonus gelir.
- `EnumMap` thread-safe değildir; eşzamanlı erişim gerekiyorsa `Collections.synchronizedMap` ile sarmalanmalıdır.
- Her iki yapı da sadece belirttikleri enum tipinin sabitlerini kabul eder; farklı bir enum sabiti eklemeye çalışmak derleme zamanında `ClassCastException`'a değil, generic tip uyuşmazlığına yol açar.

## Denemek için

```
jshell> enum Gun { PZT, SAL, CRS }
jshell> var m = new java.util.EnumMap<Gun,String>(Gun.class)
jshell> m.put(Gun.SAL, "toplanti")
jshell> m.put(Gun.PZT, "planlama")
jshell> m
```

`m`'nin içeriğini bastırdığınızda `PZT` girdisinin `SAL`'dan önce göründüğünü, yani eklenme sırasından bağımsız olarak `ordinal()` sırasına göre dizildiğini gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumMap.html]
