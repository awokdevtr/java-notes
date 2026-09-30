# Arayüzlerde default metotlar

Java 8 öncesinde bir arayüze yeni bir metot eklemek, o arayüzü implement eden her sınıfı derleme hatasına düşürürdü; yayınlanmış bir API'ye geriye dönük uyumlu şekilde davranış eklemenin yolu yoktu. `default` anahtar kelimesi, arayüz metotlarının gövde içermesine izin vererek bu sorunu çözer: implement eden sınıflar metodu override etmek zorunda kalmadan varsayılan davranışı devralır.

## Temel kullanım

```java
interface Greeter {
    default String greet(String name) {
        return "Merhaba, " + name;
    }
}

class FormalGreeter implements Greeter {
    @Override
    public String greet(String name) {
        return "Sayın " + name + ", hoş geldiniz.";
    }
}

class SimpleGreeter implements Greeter {}
```

`SimpleGreeter`, `greet` metodunu hiç yazmadan `Greeter` arayüzündeki `default` gövdeyi olduğu gibi kullanır; `FormalGreeter` ise aynı imzayı override ederek kendi davranışını tanımlar. Bir `default` metot yalnızca implement eden sınıf onu override etmediğinde devreye girer, yani somut bir sınıftaki metot her zaman arayüzdeki varsayılana önceliklidir.

## Elmas problemi ve `Interface.super`

```java
interface A {
    default String id() {
        return "A";
    }
}

interface B {
    default String id() {
        return "B";
    }
}

class C implements A, B {
    @Override
    public String id() {
        return A.super.id() + B.super.id();
    }
}
```

Bir sınıf aynı imzaya sahip `default` metoda sahip iki arayüzü implement ettiğinde derleyici hangi gövdenin kullanılacağına kendiliğinden karar veremez ve `C`'nin `id()`'yi açıkça override etmesini zorunlu kılar. `InterfaceName.super.metot()` söz dizimi, override içinden belirli bir arayüzün `default` gövdesine erişmeyi sağlar; bu, sınıf kalıtımındaki `super.metot()` çağrısının arayüz versiyonudur.

## Ne zaman işe yarar

- Mevcut bir arayüze, tüm implementasyonları bozmadan yeni bir yetenek eklemek gerektiğinde (ör. `Collection.stream()` Java 8'de böyle eklendi).
- Birden fazla implementasyonun ortak paylaşacağı yardımcı davranışı, her sınıfta tekrar yazmak yerine arayüzde merkezileştirmek istendiğinde.
- Bir sınıf ile üst sınıftan gelen somut metot her zaman arayüzdeki `default` metodu ezer; bu sıralamayı bilmeden `default` metotlara güvenmek yanlış varsayımlara yol açabilir.

## Denemek için

```
jshell> interface A { default String id() { return "A"; } }
jshell> interface B { default String id() { return "B"; } }
jshell> class C implements A, B { public String id() { return A.super.id() + B.super.id(); } }
jshell> new C().id()
```

`C` sınıfındaki `id()` override'ı kaldırılıp deney tekrarlanırsa, derleyici "sınıf A ve B'den çakışan varsayılan metotları devralıyor" hatası verir; bu, elmas probleminin derleme zamanında nasıl zorunlu çözüme bağlandığını gösterir.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html]
