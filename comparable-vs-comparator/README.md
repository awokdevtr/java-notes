# Comparable vs Comparator

Bir nesne koleksiyonunu sıralamak istediğinizde Java iki farklı yol sunar: sınıfın kendi "doğal sırasını" tanımlayan `Comparable`, ya da sıralama mantığını sınıfın dışında tutan `Comparator`. İkisini karıştırmak `ClassCastException` fırlatan ya da beklenmedik sırada sonuç veren koda yol açabilir; hangisinin ne zaman kullanılacağını bilmek bu hataları baştan önler.

## Comparable ile doğal sıra

```java
public class Calisan implements Comparable<Calisan> {
    String ad;
    int maas;

    Calisan(String ad, int maas) {
        this.ad = ad;
        this.maas = maas;
    }

    @Override
    public int compareTo(Calisan other) {
        return Integer.compare(this.maas, other.maas);
    }
}
```

`Comparable<T>` arayüzü tek bir `compareTo(T other)` metodu tanımlar ve sınıfın kendisi tarafından uygulanır; bu, o sınıfın "doğal" (`natural`) sırasıdır ve tipik olarak yalnızca bir tane olur. Dönüş değeri negatifse çağıran nesne daha küçük, pozitifse daha büyük, sıfırsa eşit kabul edilir. `Collections.sort(liste)` ya da `TreeSet` gibi sıralı koleksiyonlar, başka bir sıralama belirtilmediğinde bu metodu kullanır.

## Comparator ile dışsal sıralama

```java
Comparator<Calisan> adaGore = Comparator.comparing(c -> c.ad);
Comparator<Calisan> maasaGoreTersten =
        Comparator.comparingInt((Calisan c) -> c.maas).reversed();

liste.sort(adaGore.thenComparing(maasaGoreTersten));
```

`Comparator<T>`, sınıfın dışında tanımlanan bağımsız bir sıralama stratejisidir ve aynı tip için birden fazla `Comparator` oluşturulabilir. `Comparator.comparing`, `reversed()` ve `thenComparing()` gibi `default`/`static` metotlar, birden fazla alana göre sıralamayı zincirleme (`method chaining`) ile okunaklı şekilde ifade etmeyi sağlar. `Calisan` sınıfının kaynak koduna dokunmadan istediğiniz kadar farklı sıralama tanımlayabilirsiniz.

## Ne zaman ise yarar

- Bir sınıfın tek, mantıklı bir "varsayılan" sırası varsa (örneğin `Integer`'ın sayısal sırası) `Comparable` uygulayın.
- Aynı nesneyi farklı bağlamlarda farklı kriterlere göre sıralamanız gerekiyorsa (ada göre, maaşa göre, tarihe göre) `Comparator` kullanın; sınıfı değiştirmenize gerek kalmaz.
- `compareTo` ile `equals` tutarsızsa (`x.compareTo(y) == 0` iken `x.equals(y)` `false`), bu durum `TreeSet`/`TreeMap` gibi sıralı koleksiyonlarda eleman kaybına yol açabilir; dökümantasyon bu tutarlılığı önerir.

## Denemek için

```
jshell> var liste = new java.util.ArrayList<>(java.util.List.of("muz", "elma", "kiraz"))
jshell> liste.sort(java.util.Comparator.reverseOrder())
jshell> liste
```

`String` zaten `Comparable` olduğundan `reverseOrder()` doğal sırayı tersine çevirir ve sonuç `[muz, kiraz, elma]` olur — sınıfın kendisine dokunmadan sıralama davranışı değiştirilmiş olur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html]
