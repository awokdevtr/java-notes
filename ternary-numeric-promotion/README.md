# Ternary Operatöründe Sayısal Tip Yükseltme

`?:` operatörünün sonuç tipi, iki operandın tipine bakılarak derleme zamanında belirlenir; bu süreç göründüğünden daha karmaşıktır. Operandlardan biri `int` gibi ilkel bir sayısal tip, diğeri `Integer` gibi kutulanmış karşılığıysa derleyici ikili sayısal yükseltme (`binary numeric promotion`) uygular ve sonucu ilkel tipe indirger. Bu kural hem beklenmeyen tip dönüşümlerine hem de hiç beklenmeyen bir koşulda `NullPointerException` fırlatılmasına yol açabilir.

## Operand Tipleri Farklıysa Otomatik Genişletme

```java
int a = 10;
double b = 20.5;
System.out.println(true ? a : b); // 10.0 yazdirir, 10 degil
```

Burada bir operand `int`, diğeri `double` olduğu için derleyici ikisini ortak tipe, yani `double`'a yükseltir. Seçilen dal `a` olsa da sonuç `10.0` olarak basılır, çünkü ifadenin statik tipi her iki operanda bakılarak belirlenir, sadece seçilen dala değil. Bu davranış `+` gibi aritmetik operatörlerdeki sayısal yükseltmeyle aynı mantığı izler.

## Kutulanmış Tip ile Unboxing Kaynaklı NPE

```java
Integer count = null;
int threshold = 5;
boolean useThreshold = false;

System.out.println(useThreshold ? threshold : count); // NullPointerException
```

Operandlardan biri `int`, diğeri `Integer` olunca sonuç tipi `int`'e yükseltilir; bu da seçilen dal `count` olduğunda `Integer`'ın `unboxing` edilmesini gerektirir. `count` `null` olduğu için bu unboxing işlemi `NullPointerException` fırlatır — oysa kod `count != null` kontrolü içermediği için hata ilk bakışta mantıksız görünür. Çözüm, ya iki operandı da `Integer` tutmak ya da seçimden önce `null` kontrolü yapmaktır.

## Dikkat edilmesi gerekenler

- Operandlardan biri ilkel sayısal tip, diğeri onun kutulanmış karşılığıysa sonuç tipi ilkel tipe döner ve `null` olan operand seçildiğinde unboxing `NPE`'si oluşur.
- İki operand da referans tipiyse (örneğin ikisi de `Integer`) yükseltme uygulanmaz, sonuç `null` olabilir ve güvenlidir.
- `System.out.println(cond ? x : y)` gibi ifadelerde karışıklığı önlemek için operandları aynı tipte tutmak veya gerekirse açıkça `cast` etmek daha güvenlidir.

## Denemek için

```
jshell> Integer count = null
jshell> int threshold = 5
jshell> System.out.println(false ? threshold : count)
```

Bu üç satırı sırayla çalıştırdığınızda son satırın `NullPointerException` fırlattığını görürsünüz, oysa koşul `false` olduğu için sezgisel olarak `count`'un hiç değerlendirilmeyeceği düşünülebilir.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.25]
