# clone() ve Cloneable Tuzakları

Bir nesnenin "kopyasını" almak göründüğü kadar basit değildir. `Object.clone()` metodu, işaretleyici (marker) bir arayüz olan `Cloneable`'a güvenir ve varsayılan olarak yalnızca *shallow copy* (sığ kopya) üretir: referans tipi alanlar kopyalanmaz, aynı nesneye işaret etmeye devam eder. Bu davranışı bilmeden kullanmak, orijinal nesneyle kopyasının birbirini arkadan etkilediği ince hatalara yol açar.

## Varsayılan davranış: shallow copy

```java
class Takim implements Cloneable {
    List<String> oyuncular = new ArrayList<>();

    @Override
    public Takim clone() throws CloneNotSupportedException {
        return (Takim) super.clone();
    }
}

Takim a = new Takim();
a.oyuncular.add("Ali");

Takim b = a.clone();
b.oyuncular.add("Veli");

System.out.println(a.oyuncular); // [Ali, Veli]
```

`super.clone()`, alan alan bitwise bir kopya oluşturur; `oyuncular` referansının kendisi kopyalanır, işaret ettiği `ArrayList` nesnesi kopyalanmaz. Sonuç olarak `b` üzerinde yapılan değişiklik `a`'yı da etkiler. `Cloneable` uygulanmazsa `super.clone()` çağrısı `CloneNotSupportedException` fırlatır; bu da `clone()`'u checked exception fırlatan tuhaf bir sözleşme haline getirir.

## Deep copy için elle müdahale

```java
@Override
public Takim clone() throws CloneNotSupportedException {
    Takim kopya = (Takim) super.clone();
    kopya.oyuncular = new ArrayList<>(this.oyuncular);
    return kopya;
}
```

Gerçek bir *deep copy* için her mutable referans alanın kendisinin de klonlanması ya da yeni bir koleksiyona kopyalanması gerekir. İç içe mutable nesneler varsa bu işlem her katmanda tekrarlanmalıdır; aksi halde sığ kopyadan kalan paylaşılan referanslar sessizce kalır.

## Dikkat edilmesi gerekenler

- `Cloneable`, hiçbir metot bildirmeyen bir işaretleyici arayüzdür; asıl sözleşmeyi `Object.clone()`'un dokümantasyonu tanımlar, bu da API'yi öğrenmesi zor hale getirir.
- `final` alanlar `clone()` içinde yeniden atanamaz, bu yüzden `final` referans alanlara sahip sınıflarda deep copy genellikle kopya constructor veya statik factory ile çözülür.
- Effective Java gibi kaynaklar genellikle `Cloneable`'ı tamamen atlayıp kopya constructor (`new Takim(mevcut)`) veya kopya factory metodu önerir; bu yaklaşım checked exception ve cast gerektirmez.

## Denemek için

```
jshell> import java.util.*
jshell> class T implements Cloneable { List<String> l = new ArrayList<>(); public T clone() throws CloneNotSupportedException { return (T) super.clone(); } }
jshell> T a = new T(); a.l.add("x");
jshell> T b = a.clone(); b.l.add("y"); System.out.println(a.l);
```

`a.l`'in `[x, y]` olarak yazdırıldığını, yani `b` üzerindeki eklemenin `a`'yı da etkilediğini göreceksiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#clone()]
