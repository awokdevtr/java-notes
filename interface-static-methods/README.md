# Arayüzlerde static metotlar

Java 8 öncesinde bir arayüzle ilgili yardımcı/fabrika metotları arayüzün kendisinde barınamazdı, çünkü arayüzler gövdeli metot içeremezdi; bu yüzden `Collections` (`Collection` için), `Executors` (`Executor` için) gibi ayrı "companion" sınıflar doğdu. `static` metotların arayüz içine izin verilmesiyle bu yardımcı davranış, ait olduğu soyutlamanın yanında tanımlanabilir hale geldi.

## Temel kullanım

```java
interface Oran {

    double hesapla(double miktar);

    static Oran sabit(double yuzde) {
        return miktar -> miktar * yuzde / 100;
    }

    static Oran sifir() {
        return miktar -> 0.0;
    }
}
```

`sabit` ve `sifir`, `Oran` arayüzünün fabrika metotlarıdır ve her zaman `Oran.sabit(18)` şeklinde arayüz adı üzerinden çağrılır. `java.util.Comparator.naturalOrder()` ve `List.of(...)` da aynı deseni kullanır: nesne örneği oluşturmadan, ilgili arayüzle doğrudan ilişkili bir yardımcı davranış sunar.

## Örnekler tarafından miras alınmaz

```java
class SabitOran implements Oran {
    @Override
    public double hesapla(double miktar) {
        return miktar * 0.18;
    }
}
```

`SabitOran`'ın bir örneği `Oran.sabit(18)`'e erişemez; `new SabitOran().sabit(18)` gibi bir çağrı derleme hatası verir. `static` arayüz metotları instance metotlarının aksine implement eden sınıflara miras geçmez, bir alt arayüz tarafından da override edilemez — tamamen arayüzün kendi ad alanına ait kalırlar.

## Dikkat edilmesi gerekenler

- Arayüzün sözleşmesine (instance API'sine) dahil değildir; sadece ilgili tipe dair fabrika veya yardımcı metot sağlamak içindir.
- Bir sınıf veya alt arayüz bu metodu override edemez; aynı imzayla yeniden tanımlamak override değil, tamamen bağımsız yeni bir `static` metottur.
- `private` metotlardan farklı olarak `public`'tir ve arayüz dışından çağrılabilir, ama her zaman `ArayuzAdi.metot(...)` biçiminde.

## Denemek için

```
jshell> interface Oran { double hesapla(double m); static Oran sifir() { return m -> 0.0; } }
jshell> Oran o = Oran.sifir()
jshell> o.hesapla(100)
```

`class X implements Oran { public double hesapla(double m) { return m; } }` tanımlayıp `new X().sifir()` çağırmayı denerseniz, derleyici `sifir()` metodunun `X` örneği üzerinden görünür olmadığını belirten bir hata verir.

[Kaynak: https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-interface-methods.html]
