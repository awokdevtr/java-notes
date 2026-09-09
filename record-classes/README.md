# record classes

Sadece veri taşımak için yazılan bir sınıfın klasik hâli hep aynı kalabalığı getirir: `private final` alanlar, bir constructor, alan sayısı kadar getter, sonra elle yazılmış (ya da IDE'nin ürettiği) `equals()`, `hashCode()` ve `toString()`. Bu kod hem uzun hem de bakımı zor — bir alan eklediğinizde dört yeri birden güncellemeyi unutmak kolay. `record`, değişmez veri taşıyıcıları için bu kalıbı dile gömerek tek satırda tanımlanmasını sağlar.

## Temel kullanım

```java
public record Point(int x, int y) {}

Point p1 = new Point(3, 4);
Point p2 = new Point(3, 4);

System.out.println(p1);          // Point[x=3, y=4]
System.out.println(p1.x());      // 3
System.out.println(p1.equals(p2)); // true
```

`record Point(int x, int y)` bildirimi tek başına bir constructor, `x()` ve `y()` erişimci metotları, ve alanların tamamına dayalı `equals()`, `hashCode()`, `toString()` üretir. Alanlara `getX()` değil `x()` şeklinde erişilir — bu, `record`'un kendi imzasıdır. Bütün alanlar örtük olarak `private final`'dır, yani bir `record` her zaman değişmezdir (`immutable`).

## Compact constructor ile doğrulama

```java
public record Range(int min, int max) {
    public Range {
        if (min > max) {
            throw new IllegalArgumentException("min max'tan buyuk olamaz");
        }
    }
}
```

Parametre listesi tekrar yazılmadan tanımlanan bu `compact constructor`, alanlar atanmadan önce çalışır ve doğrulama ya da normalizasyon eklemek için kullanılır. Gövde içinde `this.min = min` yazmaya gerek yoktur; derleyici atamaları constructor'ın sonuna otomatik ekler. `record`'a normal bir metot da eklenebilir, ama alan sayısını ya da türünü değiştiremezsiniz.

## Dikkat edilmesi gerekenler

- Bir `record` başka bir sınıftan `extends` edemez (örtük olarak `java.lang.Record`'u genişletir), ama arayüz uygulayabilir.
- Alanlar `final` olduğu için `record` doğası gereği thread-safe bir veri taşıyıcıdır; ancak alan olarak tuttuğu nesne (örneğin bir `List`) kendi içinde değişebilir kalabilir.
- Sadece veri taşıyan, kimliği değil değeri önemli olan sınıflar için uygundur; davranış ağırlıklı ya da mutable durumu olan sınıflar için klasik `class` daha doğru seçimdir.

## Denemek için

```
jshell> record Point(int x, int y) {}
jshell> var a = new Point(1, 2)
jshell> var b = new Point(1, 2)
jshell> a.equals(b)
```

`a.equals(b)` çağrısının `true` döndüğünü, oysa klasik bir sınıfta `==` karşılaştırması yapılmadıkça bunun `false` olacağını görürsünüz — `record` alan bazlı `equals()`'i sizin için üretmiştir.

[Kaynak: https://docs.oracle.com/en/java/javase/17/language/records.html]
