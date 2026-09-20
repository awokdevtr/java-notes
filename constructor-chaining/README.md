# constructor-chaining

Bir sınıfın birden fazla constructor'ı olduğunda, her birinde aynı alan atamalarını ve doğrulamaları tekrar etmek hem kod tekrarına hem de tutarsızlık riskine yol açar. Biri güncellenip diğeri unutulduğunda nesne geçersiz bir durumda oluşturulabilir. Java, `this()` ve `super()` çağrılarıyla bir constructor'ın başka bir constructor'ı çağırmasına izin vererek bu tekrarı ortadan kaldırır.

## this() ile aynı sınıf içinde zincirleme

```java
public class Rectangle {
    private final double width;
    private final double height;

    public Rectangle(double width, double height) {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Kenarlar pozitif olmalı");
        }
        this.width = width;
        this.height = height;
    }

    public Rectangle(double side) {
        this(side, side);
    }
}
```

`this(side, side)` çağrısı, tek parametreli constructor'ın işi tam parametreli constructor'a devretmesini sağlar; doğrulama mantığı tek bir yerde kalır. `this()` çağrısı bir constructor gövdesinde varsa mutlaka ilk satır olmak zorundadır, çünkü derleyici nesnenin önce tam olarak inşa edilmesini garanti eder.

## super() ile üst sınıfa zincirleme

```java
public class Shape {
    protected final String name;

    public Shape(String name) {
        this.name = name;
    }
}

public class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        super("Circle");
        this.radius = radius;
    }
}
```

Her constructor, gövdesinde açıkça `this()` veya `super()` çağırmazsa derleyici otomatik olarak üst sınıfın parametresiz `super()` constructor'ını satır başına ekler. `Shape` gibi parametresiz constructor'ı olmayan bir üst sınıftan türetilen sınıflar, alt sınıf constructor'ında uygun `super(...)` çağrısını açıkça yazmak zorundadır; aksi halde derleme hatası alınır.

## Dikkat edilmesi gerekenler

- Bir constructor içinde hem `this()` hem `super()` aynı anda çağrılamaz; ikisi de yalnızca ilk satırda olabilir, dolayısıyla biri diğerini zaten dolaylı olarak tetikler.
- Zincirleme döngüsel olamaz: `A(int)` içinde `this()` ile `A()`'yı çağırıp `A()` içinde tekrar `A(int)`'i çağırmak derleme zamanında tespit edilip hataya yol açar.
- Alan başlatıcıları (`field initializer`) ve instance initializer bloklar, zincirlenen constructor'ın gövdesi çalışmadan hemen önce, her constructor çağrısında yeniden çalışır; bu yüzden `this()` zincirlemesinde bile başlatıcı kodu birden fazla kez tekrarlanmaz, yalnızca zincirin sonunda bir kez asıl gövdeler çalışır.

## Denemek için

```
jshell> class A { A() { System.out.println("A()"); } A(int x) { this(); System.out.println("A(int)=" + x); } }
jshell> new A(5)
```

Çıktıda önce `A()` sonra `A(int)=5` satırlarının basıldığını görürsünüz; bu da `this()` çağrısının, kalan gövde çalışmadan önce tamamlandığını kanıtlar.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/IandI/super.html]
