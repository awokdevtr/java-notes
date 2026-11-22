# Comparator.nullsFirst / nullsLast

Bir listeyi `null` değerler içerebilecek bir alana göre sıralamaya çalıştığınızda klasik `Comparator`, `compareTo` ya da `compare` içinde `null` ile karşılaştığı anda `NullPointerException` fırlatır. Sıralama mantığının her yerine `if (a == null) ...` kontrolü eklemek yerine, `Comparator.nullsFirst`/`nullsLast` bu null kontrolünü sarmalayıcı bir katman olarak ekler.

## Temel kullanım

```java
List<String> names = new ArrayList<>(List.of("Ayşe", null, "Can", null, "Berk"));

names.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(names); // [null, null, Ayşe, Berk, Can]

names.sort(Comparator.nullsLast(Comparator.naturalOrder()));
System.out.println(names); // [Ayşe, Berk, Can, null, null]
```

`nullsFirst`/`nullsLast`, verdiğiniz asıl `Comparator`'ı sarar: `null` değerleri kendisi yakalar ve sıralamanın başına ya da sonuna yerleştirir, `null` olmayan değerleri ise olduğu gibi size verdiğiniz `comparator`'a bırakır. İçteki `comparator` hiçbir zaman `null` görmez — bu yüzden `Comparator.naturalOrder()` gibi `null` konusunda hiçbir koruması olmayan bir karşılaştırıcıyı güvenle kullanabilirsiniz.

## thenComparing ile alan bazlı sıralama

```java
record Kisi(String ad, Integer yas) {}

List<Kisi> kisiler = new ArrayList<>(List.of(
    new Kisi("Deniz", null),
    new Kisi("Deniz", 30),
    new Kisi("Ali", 25)
));

kisiler.sort(
    Comparator.comparing(Kisi::ad)
        .thenComparing(Kisi::yas, Comparator.nullsLast(Comparator.naturalOrder()))
);
```

Burada `nullsFirst`/`nullsLast` sadece en dıştaki `comparator` değil, `thenComparing` zincirinin herhangi bir halkasında da kullanılabilir — her alan kendi `null` politikasını bağımsız seçebilir. `ad` alanında `null` yoksa düz `Comparator::compare` yeterlidir, ama `yas` gibi `null` olabilen bir alanda bu sarmalayıcı olmadan zincir ilk `null`'da patlar.

## Dikkat edilmesi gerekenler

- `nullsFirst`/`nullsLast`'a `null` comparator geçmek de geçerlidir (`Comparator.nullsFirst(null)`); bu durumda `null` olmayan tüm elemanlar birbirine eşit sayılır, sıralama sadece `null`'ları ayırır.
- `reversed()` çağrısı `null` konumunu da ters çevirir — `nullsFirst(...).reversed()`, `null`'ları sona, değerleri ise azalan sırada başa alır. Beklenmedik sonuç almamak için önce içteki `comparator`'ı `reversed()` yapıp sonra `nullsFirst`/`nullsLast` ile sarmak daha öngörülebilirdir.
- Bu metotlar sadece `Comparator` seviyesinde çalışır; `TreeMap`/`TreeSet` gibi doğal sıralamaya (`Comparable`) dayanan koleksiyonlarda anahtar olarak `null` kullanmak hâlâ ayrı bir konudur ve genelde desteklenmez.

## Denemek için

```
jshell> import java.util.*
jshell> var list = new ArrayList<>(List.of("b", null, "a"))
jshell> list.sort(Comparator.nullsFirst(Comparator.naturalOrder()))
jshell> list
```

`list` çıktısında `null`'ın listenin başına taşındığını, `Comparator.naturalOrder()`'ı tek başına çağırsaydınız ise bu satırın `NullPointerException` ile patlayacağını karşılaştırarak görebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html#nullsFirst(java.util.Comparator)]
