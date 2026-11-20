# enum-values-array-cost

Her `enum` türü derleyici tarafından örtük bir `static values()` metoduyla donatılır. Bu metot pratik görünür ama her çağrıldığında dizinin tam bir kopyasını oluşturur; döngü içinde ya da sık çağrılan bir yolda kullanılırsa fark edilmeyen ama gerçek bir performans maliyeti ortaya çıkar.

## Gizli kopyalama maliyeti

```java
public enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

for (int i = 0; i < 1_000_000; i++) {
    for (Day d : Day.values()) { // her iterasyonda yeni dizi
        // ...
    }
}
```

JVM, her `enum` türü için sabitleri tutan `private static final` bir dizi saklar; `values()` her çağrıda bu dizinin `clone()`'unu döndürür, çünkü diziler Java'da mutable'dır ve dışarıya verilen referans üzerinden iç dizinin değiştirilmesine izin verilemez. Dış döngüde bir milyon kez `Day.values()` çağırmak bir milyon kez yedi elemanlı dizi kopyalama anlamına gelir; eleman sayısı büyüdükçe veya çağrı sıklığı arttıkça bu maliyet toplanır.

## Kopyalamadan kaçınma

```java
public enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN;

    private static final Day[] VALUES = values(); // bir kez kopyala, sakla
}

for (Day d : Day.VALUES) { // artık kopyalama yok
    // ...
}
```

Sık çağrılan bir yolda `values()` sonucunu bir kez statik bir alana atayıp sonrasında o alanı kullanmak kopyalamayı tek seferle sınırlar. Alternatif olarak sabit koleksiyonu dışarı vermek gerekiyorsa `EnumSet.allOf(Day.class)` da benzer bir önceden-hesaplanmış maliyetle çalışır ama döndürdüğü `Set` değiştirilemez bir görünüm değildir, yine de referans paylaşımı `values()`'ten farklıdır.

## Dikkat edilmesi gerekenler

- `VALUES` alanını `public` yaparsanız çağıranlar diziyi değiştirebilir ve bu değişiklik tüm türü etkiler; alanı `private` tutup dışarıya `List.of(VALUES)` ya da `Collections.unmodifiableList` ile değiştirilemez bir görünüm sunmak daha güvenlidir.
- Küçük enum'larda (birkaç sabitli) bu maliyet genelde göz ardı edilebilir; sorun sıcak döngülerde (`hot path`) veya çok elemanlı enum'larda belirginleşir.

## Denemek için

```
jshell> enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }
jshell> Day.values() == Day.values()
```

Her çağrı farklı bir dizi nesnesi döndürdüğü için sonucun `false` olduğunu, yani her seferinde yeni bir kopya üretildiğini görürsünüz.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.3]
