# record patterns

`record` ile taşınan verinin alanlarına ulaşmak için klasik yol, önce `instanceof` ile tipi kontrol etmek, sonra cast etmek, sonra da erişimci metotları tek tek çağırmaktır. Bir `record` içinde başka bir `record` varsa bu zincir daha da uzar. Java 21 ile gelen `record pattern`, bir `record`'u `instanceof` veya `switch` içinde doğrudan parçalarına ayırıp alanları tek adımda değişkenlere bağlar.

## Temel kullanım

```java
record Point(int x, int y) {}

Object obj = new Point(3, 4);

if (obj instanceof Point(int x, int y)) {
    System.out.println(x + y);   // 7
}
```

`Point(int x, int y)` deseni, `obj` gerçekten bir `Point` ise hem tip kontrolünü yapar hem de alanları `x` ve `y` adlı yerel değişkenlere `deconstruct` eder. Ayrı bir cast ya da `p.x()`/`p.y()` çağrısına gerek kalmaz; bu alanlara bloğun içinde doğrudan ilkel değişkenler gibi erişilirsiniz.

## İç içe record'larda ayrıştırma

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

static String describe(Object obj) {
    return switch (obj) {
        case Line(Point(var x1, var y1), Point(var x2, var y2)) ->
            "(%d,%d) -> (%d,%d)".formatted(x1, y1, x2, y2);
        default -> "bilinmeyen";
    };
}
```

`switch` içindeki `Line(Point(var x1, var y1), Point(var x2, var y2))` deseni, iç içe geçmiş iki `record`'u tek satırda tamamen açar; `line.start().x()` gibi zincirleme çağrılara gerek kalmaz. `var` kullanımı burada her alanın tipini derleyiciye bırakır, ama açık tip adı yazmak da (`int x1` gibi) geçerlidir.

## Dikkat edilmesi gerekenler

- Record pattern yalnızca `record` tipleriyle çalışır; normal bir `class` için bu şekilde ayrıştırma yapılamaz.
- Desendeki alan sayısı ve sırası, `record`'un bildirimindeki `canonical constructor` ile tam eşleşmelidir; eksik veya fazla alan derleme hatasıdır.
- `switch` içinde kullanıldığında `instanceof` pattern matching'deki `flow scoping` kuralları geçerlidir: değişkenler sadece eşleşen dalda görünürdür.

## Denemek için

```
jshell> record Point(int x, int y) {}
jshell> Object obj = new Point(5, 9)
jshell> if (obj instanceof Point(int x, int y)) System.out.println(x * y)
```

Son satırın `45` yazdırdığını görürsünüz — `obj instanceof Point(int x, int y)` ifadesi tip kontrolünü yaparken `x` ve `y`'yi de aynı anda değişkenlere bağlamıştır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/language/record-patterns.html]
