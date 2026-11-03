# Math.clamp ile Değeri Bir Aralığa Sıkıştırma

Bir değeri belirli bir `[min, max]` aralığında tutmak için uzun zamandır `Math.max(min, Math.min(max, value))` gibi iç içe çağrılar yazılır. Bu ifade okunması zor olmasının yanında, `min` ve `max` argümanlarının sırası ters yazıldığında derleyici hiçbir uyarı vermez ve hata sessizce kalır. Java 21, bu kalıbı `Math.clamp` ile tek, niyeti açık bir çağrıya indirir.

## `Math.max`/`Math.min` Zincirinin Yerini Alması

```java
int score = 137;
int clamped = Math.max(0, Math.min(100, score));   // eski kalıp
int clampedNew = Math.clamp(score, 0, 100);         // Java 21+
System.out.println(clampedNew);   // 100
```

`Math.clamp(value, min, max)`, `value` aralığın içindeyse olduğu gibi döner; aralığın altındaysa `min`, üstündeyse `max` döner. `int`, `long`, `float` ve `double` için overload'ları vardır; tamsayı overload'larında `min` ve `max` sırasıyla `long` tipindedir, bu yüzden `int` çağrılarda da sınırlar örtük olarak genişletilip geri daraltılır.

## Geçersiz Aralıkta `IllegalArgumentException`

```java
Math.clamp(5, 10, 1);   // min > max -> IllegalArgumentException
```

`min` değeri `max`'tan büyükse metot sessizce yanlış bir sonuç döndürmek yerine `IllegalArgumentException` fırlatır. Bu, `Math.max(min, Math.min(max, value))` kalıbının sınırlar ters yazıldığında sessizce (ve yanlış) bir sonuç üretmesinden farklıdır; `clamp` hatayı derleme zamanında değil ama en azından çalışma zamanında hemen açığa çıkarır.

## Dikkat edilmesi gerekenler

- `float`/`double` overload'ları `NaN` içeren `min`/`max`/`value` için de `IllegalArgumentException` fırlatır; `NaN` aralığın parçası olarak kabul edilmez.
- `int`/`long` overload'larında sınırlar `long` tipindedir, bu yüzden `int` taşması riski olmadan tüm `int` değer aralığı güvenle sıkıştırılabilir.
- Java 21 öncesi sürümlerde bu metot yoktur; eski kod tabanlarında hâlâ `Math.max`/`Math.min` zinciri veya `Guava`'nın benzer yardımcıları kullanılır.

## Denemek için

```
jshell> Math.clamp(137, 0, 100)
jshell> Math.clamp(-5, 0, 100)
jshell> Math.clamp(5, 10, 1)
```

İlk iki satır üst ve alt sınıra sıkıştırmayı gösterir, üçüncü satır geçersiz `min > max` aralığında fırlatılan `IllegalArgumentException`'ı gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Math.html#clamp(long,long,long)]
