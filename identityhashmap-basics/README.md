# IdentityHashMap

Normal bir `HashMap`, anahtarları `equals()` ve `hashCode()` sözleşmesine göre karşılaştırır: içerikleri aynı olan iki farklı nesne aynı anahtar sayılır. Bazı durumlarda bu istenmez -- örneğin bir serileştirme (serialization) ya da derin kopyalama algoritmasında "bu nesneyi daha önce ziyaret ettim mi" sorusunun cevabı içeriğe değil, tam olarak aynı referansa bakmalıdır. `IdentityHashMap`, anahtar karşılaştırmasını `equals()`/`hashCode()` yerine `==` ve `System.identityHashCode()` üzerinden yapan bir `Map` uygulamasıdır.

## Referans eşitliği ile anahtarlama

```java
record Point(int x, int y) {}

Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);

Map<Point, String> normal = new HashMap<>();
normal.put(p1, "ilk");
normal.put(p2, "ikinci");
System.out.println(normal.size());

Map<Point, String> identity = new IdentityHashMap<>();
identity.put(p1, "ilk");
identity.put(p2, "ikinci");
System.out.println(identity.size());
```

`record`'un otomatik ürettiği `equals()` içerik karşılaştırması yaptığı için `normal` haritasında `p1` ve `p2` aynı anahtar sayılır ve harita tek girdiye düşer, boyut `1` olur. `IdentityHashMap` ise anahtarın `equals()`'ını hiç çağırmaz; `p1` ve `p2` farklı referanslar olduğundan iki ayrı giriş olarak tutulur ve boyut `2` olur.

## Nesne grafiklerinde döngü tespiti

```java
Map<Object, Boolean> visited = new IdentityHashMap<>();

void traverse(Object node) {
    if (visited.put(node, Boolean.TRUE) != null) {
        return;
    }
    // node'un alanlarini gez
}
```

Bu kalıp, derin kopyalama veya graph serialization gibi algoritmalarda sıkça görülür: amaç "bu tam nesneyi işledim mi" sorusunu `equals()` tanımından tamamen bağımsız, saf referans kimliğiyle sormaktır. `visited.put()` zaten ziyaret edilmiş bir referans için `null` olmayan bir değer döndürür ve döngüyü keser.

## Dikkat edilmesi gerekenler

- `IdentityHashMap`, `Map` arayüzünün genel sözleşmesini kasıtlı olarak ihlal eder; `equals()`/`hashCode()`'a dayanan kodla karıştırılmamalıdır.
- Thread-safe değildir ve eşzamanlı erişimde harici senkronizasyon gerektirir.
- Anahtar olarak kullanılan nesnenin kendi `equals()`/`hashCode()` metotları hiç çağrılmaz; bu yüzden yanlışlıkla genel amaçlı bir `Map` yerine konulursa sessizce hatalı davranışa yol açar.

## Denemek için

```
jshell> var m = new java.util.IdentityHashMap<String, Integer>()
jshell> m.put(new String("a"), 1); m.put(new String("a"), 2); m.size()
```

İki `new String("a")` çağrısı içerikçe eşit ama farklı referanslar ürettiği için `size()` değeri `1` değil `2` döner; aynı kodu bir `HashMap` ile denerseniz `equals()` eşleşmesi nedeniyle sonuç `1` olur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/IdentityHashMap.html]
