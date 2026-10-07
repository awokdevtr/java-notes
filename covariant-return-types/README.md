# Kovaryant Dönüş Tipleri (Covariant Return Types)

Bir metot override edildiğinde, alt sınıftaki versiyonun üst sınıftaki imzayla aynı dönüş tipini kullanması gerekmez. Java 5'ten itibaren alt sınıf, üst sınıfın bildirdiği dönüş tipinin bir **alt tipini** döndürebilir; buna kovaryant dönüş tipi denir. Bu, özellikle `clone()` gibi metotlarda veya fabrika metotlarında çağıranın gereksiz cast yapmasını ortadan kaldırır.

## Alt sınıfta daha dar bir dönüş tipi

```java
class Hayvan {
    Hayvan uret() { return new Hayvan(); }
}

class Kedi extends Hayvan {
    @Override
    Kedi uret() { return new Kedi(); }
}

Hayvan h = new Kedi();
Hayvan yeni = h.uret(); // çalışma zamanında Kedi.uret() çağrılır
```

`Kedi.uret()` metodu, `Hayvan.uret()`'in bildirdiği `Hayvan` dönüş tipi yerine onun alt tipi olan `Kedi`'yi döndürür ve bu hâlâ geçerli bir `override`'dır; derleyici `Kedi`'nin bir `Hayvan` olduğunu bildiği için imza uyumluluğunu kabul eder. `@Override` anotasyonu burada derleyiciye niyetin gizleme değil override olduğunu doğrulatır. Çağıran taraf `h` referansı üzerinden hâlâ `Hayvan` tipinde bir sonuç görür, ama nesnenin gerçek tipi her zaman `Kedi`'dir.

## Cast'siz kullanım avantajı

```java
class Document implements Cloneable {
    @Override
    public Document clone() {
        try {
            return (Document) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}

Document d = new Document();
Document kopya = d.clone(); // Object.clone() imzası Object döndürür ama cast gerekmez
```

`Object.clone()` dönüş tipi `Object` olduğu halde, `Document.clone()` bunu `Document` olarak daraltır; bu sayede çağıran kod `(Document) d.clone()` yazmak zorunda kalmaz. Kovaryant dönüş tipi olmasaydı her alt sınıfta `clone()` çağıran kodun kendi cast'ini yazması gerekirdi, bu da hem tekrarlı hem de `ClassCastException` riski taşıyan bir kalıp olurdu.

## Dikkat edilmesi gerekenler

- Kovaryant dönüş sadece referans tiplerinde geçerlidir; ilkel tipler (`int`, `long` gibi) arasında bu kural işlemez, dönüş tipi tam olarak aynı olmalıdır.
- Parametre tipleri kovaryant olamaz; sadece dönüş tipi daraltılabilir, aksi halde bu bir `override` değil bağımsız bir `overload` olur.
- Dönüş tipini genişletmek (üst sınıfın dönüş tipinden daha geniş bir tip döndürmek) derleme hatasıdır; sadece daraltma (alt tipe geçiş) desteklenir.
- IDE ve derleyici, `@Override` anotasyonu ile imza uyumunu kontrol eder; kovaryant dönüş yanlış yazıldığında (örneğin ilişkisiz bir tip) derleyici bunu override hatası olarak reddeder.

## Denemek için

```
jshell> class Hayvan { Hayvan uret() { return new Hayvan(); } }
jshell> class Kedi extends Hayvan { Kedi uret() { return new Kedi(); } }
jshell> Hayvan h = new Kedi()
jshell> h.uret().getClass()
```

Çıktı olarak `class Kedi` göreceksiniz; `h` değişkeninin derleme zamanı tipi `Hayvan` olsa da, `uret()` çağrısı çalışma zamanında `Kedi.uret()`'e gider ve kovaryant dönen gerçek tip korunur.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4.5]
