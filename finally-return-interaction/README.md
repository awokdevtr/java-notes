# finally ve return etkileşimi

`finally` bloğu, `try` içinde ne olursa olsun (normal dönüş, `return`, `break`, `continue` ya da fırlatılan bir istisna) her zaman çalışır. Ancak `finally` içinde de bir `return`, `break`, `continue` veya yeni bir istisna varsa, bu **ani tamamlanma (abrupt completion)** öncekini tamamen iptal eder — `try` bloğunun döndürmeye çalıştığı değer veya fırlattığı istisna sessizce kaybolur. Bu, hata ayıklaması zor, sinsi bir davranış kaynağıdır.

## `finally` içindeki `return`, `try`'daki değeri gölgeler

```java
static int calculate() {
    try {
        return 1;
    } finally {
        return 2;
    }
}

calculate(); // 2
```

`try` bloğu `1` döndürmeye karar verir, ancak metottan gerçekten çıkmadan önce `finally` çalıştırılır. `finally` içindeki `return 2` kendi ani tamamlanmasını dayattığı için `try`'ın döndürmek istediği `1` hiç geri dönmez, JVM tarafından atılır. Metodun nihai sonucu her zaman `finally`'nin verdiği değerdir.

## `finally` bir istisnayı da yutabilir

```java
static int risky() {
    try {
        throw new RuntimeException("try'dan fırlatıldı");
    } finally {
        return 42;
    }
}

risky(); // 42, istisna hiç görünmez
```

`try` bloğu bir `RuntimeException` fırlatır, fakat `finally` çalışırken karşılaşılan `return 42` bu istisnayı da bastırır. Çağıran taraf istisnayı asla yakalayamaz; metot sanki hiçbir sorun yokmuş gibi `42` döndürür. Aynı kural `finally` içinde yeni bir istisna fırlatıldığında da geçerlidir: orijinal istisna kaybolur, yalnızca `finally`'nin fırlattığı görünür.

## Dikkat edilmesi gerekenler

- `finally` içine `return`, `break` veya `continue` koymak, dilin izin verdiği ama neredeyse her zaman kaçınılması gereken bir kalıptır.
- Bu davranış derleyici hatası değildir; bazı statik analiz araçları (örn. Error Prone, SonarQube) bunu uyarı olarak işaretler.
- `finally` yalnızca temizlik (kaynak kapatma, log yazma) için kullanılmalı, akış kontrolü veya değer döndürme için kullanılmamalıdır.
- `try-with-resources` bu tuzağı büyük ölçüde ortadan kaldırır çünkü kaynak kapatma kodu elle yazılan `finally` yerine derleyici tarafından üretilir.

## Denemek için

```
jshell> int f() { try { throw new RuntimeException("x"); } finally { return 99; } }
jshell> f()
```

Beklenenin aksine istisna fırlamaz, doğrudan `99` değeri döner — `finally` içindeki `return`, `try`'da fırlatılan istisnayı sessizce yutar.

[Kaynak: https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html]
