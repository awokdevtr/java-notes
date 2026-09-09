# sealed classes

Bir sınıf hiyerarşisinde "bu arayüzü/sınıfı sadece şu belirli tipler uygulayabilir/genişletebilir" demek istediğinizde klasik çözüm ya `final` (genişletmeyi tamamen kapatır) ya da hiç kısıtlama koymamaktır (herkes uygulayabilir, `switch` ile tüm durumları kapsadığınızdan emin olamazsınız). `sealed`, Java 17 ile bu ikisi arasındaki boşluğu doldurur: hiyerarşiyi belirli bir alt küme ile sınırlar ve derleyici bu kümenin tam olarak bilinmesini garanti eder.

## Temel kullanım

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}

public final class Circle implements Shape {
    double radius;
}

public final class Rectangle implements Shape {
    double width, height;
}

public non-sealed class Triangle implements Shape {
    double base, height;
}
```

`permits` listesi izin verilen alt tipleri açıkça belirtir; listelenmeyen bir sınıf `Shape`'i uygulayamaz. Her alt tip üç durumdan birini seçmek zorundadır: `final` (hiyerarşi burada kapanır), `sealed` (kendi `permits` listesiyle daha da kısıtlanır) ya da `non-sealed` (hiyerarşiyi tekrar herkese açar). Aynı dosyada tanımlıysa `permits` yazmaya gerek yoktur, derleyici alt tipleri otomatik bulur.

## Exhaustive switch ile birlikte kullanım

```java
static double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius * c.radius;
        case Rectangle r -> r.width * r.height;
        case Triangle t -> 0.5 * t.base * t.height;
    };
}
```

`Shape` sealed olduğu için derleyici tüm alt tiplerin bilindiğini bilir; bu `switch` ifadesinde `default` dalı olmadan da derleme geçer. Yeni bir `Shape` alt tipi eklenip `permits` listesine dahil edildiğinde, onu işlemeyen her `switch` derleme hatası verir — bu da hiyerarşiye yeni tip eklerken hiçbir `case`'in unutulmamasını garanti eder.

## Ne zaman işe yarar

- Sonlu, bilinen bir varyant kümesini modellemek istediğinizde (örneğin bir AST düğüm tipi, bir sonuç/hata durumu).
- `record` ile birlikte kullanıldığında (`sealed interface Result permits Success, Failure`) cebirsel veri tipi benzeri bir yapı elde edilir.
- Kütüphane yazarken API'nizin genişletilme noktalarını kasıtlı olarak sınırlamak istediğinizde.

## Denemek için

```
jshell> sealed interface Animal permits Dog, Cat {}
jshell> final class Dog implements Animal {}
jshell> final class Cat implements Animal {}
jshell> Animal a = new Dog();
jshell> String result = switch (a) { case Dog d -> "dog"; case Cat c -> "cat"; };
jshell> System.out.println(result)
```

`default` dalı eklemeden `switch` ifadesinin derlendiğini görürsünüz. Şimdi `Cat` sınıfını `permits` listesinden çıkarıp tekrar derlemeyi deneyin; `Animal`'ı uygulayamayacağı için hata alırsınız.

[Kaynak: https://docs.oracle.com/en/java/javase/17/language/sealed-classes-and-interfaces.html]
