# java.util.Objects yardımcı metotları

`equals()` ve `hashCode()` yazarken null kontrolü genelde elle yapılır: `a == null ? b == null : a.equals(b)` gibi ifadeler her alan için tekrar tekrar yazılır ve bir `NullPointerException` unutulduğunda sessizce hataya dönüşür. `java.util.Objects`, bu tekrarı null-güvenli, tek satırlık statik metotlara indirir.

## equals, hashCode ve toString

```java
final class Nokta {
    private final Integer x;
    private final String etiket;

    Nokta(Integer x, String etiket) {
        this.x = x;
        this.etiket = etiket;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Nokta n)) return false;
        return Objects.equals(x, n.x) && Objects.equals(etiket, n.etiket);
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, etiket);
    }

    @Override
    public String toString() {
        return Objects.toString(etiket, "etiketsiz");
    }
}
```

`Objects.equals(a, b)` her iki taraf da `null` olduğunda `true`, sadece biri `null` olduğunda `true`'ya düşmeden `false` döner; `a.equals(b)` çağrısını hiç yapmaz. `Objects.hash(...)` verilen alanları bir `Object[]`'e sarıp `Arrays.hashCode` ile birleştirir; bu yüzden kısa bir sözdizimi karşılığında küçük bir kutulama ve dizi oluşturma maliyeti öder. `Objects.toString(obj, varsayilan)` ise `obj` `null` olduğunda ikinci argümanı döndürür.

## requireNonNull ile erken doğrulama

```java
class Siparis {
    private final String musteriId;

    Siparis(String musteriId) {
        this.musteriId = Objects.requireNonNull(musteriId, "musteriId null olamaz");
    }
}
```

`Objects.requireNonNull`, argüman `null` ise mesajı taşıyan bir `NullPointerException` fırlatır ve aksi halde argümanı aynen döndürür — bu da onu constructor ve setter'larda doğrudan atama ifadesinin içine yerleştirmeyi mümkün kılar. Hata, sorunun kaynağında (constructor'da) fırlatıldığı için, `musteriId` null iken nesne oluşup sorunun saatler sonra `NullPointerException` olarak başka bir yerde patlaması önlenir. `requireNonNullElse(deger, varsayilan)` ise exception fırlatmak yerine `null` durumunda bir varsayılan değer sağlar.

## Dikkat edilmesi gerekenler

- `Objects.hash(...)` varargs aldığı için her çağrıda bir dizi tahsis eder; çok sık çağrılan sıcak (hot path) kodda alanları elle birleştirmek daha performanslıdır.
- `Objects.equals` kısa devre yapar ama hâlâ taraflardan birinin `equals()` metodunu çağırır; o metodun kendisi null-güvenli ve simetrik olmalıdır.
- `requireNonNull`'un mesaj parametresi bir `Supplier<String>` da alabilir (`requireNonNullMessage(obj, () -> "...")`); mesaj pahalı üretiliyorsa bu overload'u tercih edin, aksi halde her çağrıda string birleştirme maliyeti doğar.

## Denemek için

```
jshell> import java.util.Objects
jshell> Objects.equals(null, null)
jshell> Objects.equals("a", null)
jshell> Objects.requireNonNull(null, "deger gerekli")
```

Son satır, mesajı taşıyan bir `NullPointerException` fırlatacaktır; mesajın argümanın kendisinden değil, çağrı yerinden geldiğine dikkat edin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Objects.html]
