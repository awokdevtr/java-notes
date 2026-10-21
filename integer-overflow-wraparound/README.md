# Integer Taşması ve Sessiz Sarma (Overflow / Wraparound)

Java'da `int` ve `long` aritmetiği, C gibi dillerde olduğu gibi sabit bit genişliğinde çalışır ve sınır aşıldığında hiçbir exception fırlatmaz. `Integer.MAX_VALUE`'ya `1` eklemek programı durdurmaz; sonuç sessizce `Integer.MIN_VALUE`'ya "sarar" (wraps around). Bu davranış, özellikle toplama/çarpma işlemleriyle büyüyen sayaçlarda veya hash hesaplamalarında fark edilmeyen hatalara yol açabilir.

## Sessiz Taşma Davranışı

```java
int max = Integer.MAX_VALUE;
System.out.println(max + 1);        // -2147483648
System.out.println(max * 2);        // -2
```

`int` iki'nin tümleyeni (`two's complement`) gösterimiyle saklanır; 32 bitlik sınırı aşan bir toplama veya çarpma, üst bitleri sessizce atar ve sonuç negatif tarafa "sarar". Derleyici veya `JVM` bunu bir hata olarak görmez, çünkü `JLS` (Java Language Specification) bu davranışı tanımlı kabul eder.

## Math.*Exact ile Erken Tespit

```java
try {
    int result = Math.addExact(Integer.MAX_VALUE, 1);
} catch (ArithmeticException e) {
    System.out.println("Taşma tespit edildi: " + e.getMessage());
}
```

`Math.addExact`, `Math.subtractExact`, `Math.multiplyExact` ve `Math.toIntExact` metotları, normal operatörlerin aksine taşma durumunda `ArithmeticException` fırlatır. Bu, özellikle `long`'dan `int`'e daraltma (`narrowing`) yapılan veya kullanıcı girdisiyle büyüyen toplamların hesaplandığı kodlarda taşmayı sessizce geçiştirmek yerine erken ve açıkça yakalamayı sağlar.

## Dikkat edilmesi gerekenler

- Büyük veya sınırsız büyüyebilen değerler için `int` yerine `long` ya da gerekirse `BigInteger` kullanın.
- `hashCode()` gibi bazı yerlerde taşma kasıtlı ve beklenen bir davranıştır; her taşma bir hatanın işareti değildir.
- Kullanıcı girdisinden veya dış kaynaktan gelen toplama/çarpma işlemlerinde `Math.*Exact` kullanmak, sessiz veri bozulmasını önler.

## Denemek için

```
jshell> Integer.MAX_VALUE + 1
jshell> Math.addExact(Integer.MAX_VALUE, 1)
jshell> Math.toIntExact(3000000000L)
```

İlk satır sessiz sarmayı gösterir, ikinci ve üçüncü satırlar `ArithmeticException` fırlatarak taşmayı açıkça ortaya çıkarır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Math.html#addExact(int,int)]
