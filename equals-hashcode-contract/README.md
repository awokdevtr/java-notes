# equals ve hashCode Sözleşmesi

`Object` sınıfından miras alınan `equals()` varsayılan olarak referans eşitliği (`==`) uygular; iki farklı nesne aynı alanlara sahip olsa bile birbirine eşit sayılmaz. Bir sınıfı `HashMap` anahtarı ya da `HashSet` elemanı olarak kullanmak istediğinizde bu davranış beklenmedik sonuçlar doğurur: `equals()`'i override edip `hashCode()`'u atlamak, koleksiyonların nesneyi "kaybetmesine" yol açan klasik bir hatadır.

## Sözleşmeye uygun override

```java
public final class Point {
    private final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point p)) return false;
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

Sözleşme şunu şart koşar: `a.equals(b)` `true` dönüyorsa `a.hashCode() == b.hashCode()` de doğru olmalıdır. `HashMap` ve `HashSet` önce `hashCode()` ile doğru bucket'ı bulur, sonra o bucket içindeki adaylarla `equals()` karşılaştırması yapar; iki metot tutarsızsa nesne yanlış bucket'a düşer ve asla bulunamaz.

## Dikkat edilmesi gerekenler

- `equals()` ve `hashCode()`'u her zaman birlikte override edin; biri override edilip diğeri unutulursa sözleşme bozulur.
- `hashCode()` hesaplamasında kullanılan alanlar mutable ise ve nesne `HashSet`'e eklendikten sonra değiştirilirse, nesne koleksiyon içinde "kaybolur" çünkü artık doğru bucket'ta aranmaz.
- `Objects.equals()` ve `Objects.hash()` yardımcı metotları `null` kontrolünü sizin yerinize yapar, elle yazılan karşılaştırmalardan daha az hataya açıktır.

## Denemek için

```
jshell> record P(int x, int y) {}
jshell> var s = new java.util.HashSet<P>(); s.add(new P(1, 2)); s.contains(new P(1, 2))
```

Sonucun `true` döndüğünü görürsünüz çünkü `record` sınıfları `equals()` ve `hashCode()`'u tüm bileşen alanlarına göre otomatik ve tutarlı şekilde üretir. Aynı denemeyi bu iki metodu override etmeyen sıradan bir sınıfla yaparsanız `contains()` `false` döner.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)]
