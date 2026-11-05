# serialization-basics

Bir nesneyi dosyaya yazmak, ağ üzerinden göndermek ya da bir cache'te saklamak için bayt dizisine çevirmeniz gerekir. Java'da bunun yerleşik yolu `Serializable` arayüzüdür, ama arayüz boş (metotsuz) olduğu için "neyin nasıl serileştirileceği" büyük ölçüde JVM'in kendi kurallarına bırakılır. Bu da `serialVersionUID` ve `transient` gibi ayrıntıları göz ardı edildiğinde sessizce kırılan, hata ayıklaması zor sorunlara yol açar.

## Temel kullanım

```java
import java.io.*;

class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private final String name;
    private transient String sessionToken;

    User(String name, String sessionToken) {
        this.name = name;
        this.sessionToken = sessionToken;
    }
}

try (var out = new ObjectOutputStream(new FileOutputStream("user.bin"))) {
    out.writeObject(new User("ayse", "secret-token"));
}
```

`Serializable`, sınıfa herhangi bir metot zorunluluğu getirmez; sadece JVM'e "bu sınıfın örnekleri serileştirilebilir" sinyalini verir. `ObjectOutputStream.writeObject()` çağrıldığında alanlar reflection ile okunup bayt dizisine yazılır. `transient` işaretli alanlar bu sürece hiç dahil edilmez; `sessionToken` disk üzerinde yer almaz ve geri okunduğunda `null` olarak gelir. Bu, şifre veya oturum anahtarı gibi kalıcı olmaması gereken verileri serileştirmeden hariç tutmanın standart yoludur.

## serialVersionUID uyumsuzluğu

```java
// Sınıfı ilk yazdığınızda: serialVersionUID = 1L, iki alan
// Sonra bir alan eklediniz ama serialVersionUID'yi elle sabit tuttunuz
try (var in = new ObjectInputStream(new FileInputStream("user.bin"))) {
    User u = (User) in.readObject(); // eski dosya, yeni sınıf tanımı
}
```

`serialVersionUID`'yi açıkça tanımlamazsanız derleyici onu sınıfın alanlarına, metotlarına ve arayüzlerine göre otomatik hesaplar. Sınıfı derleyen her JVM ya da `javac` sürümü bu hesaplamayı az farklı yapabilir; sonuç olarak aynı kaynak koddan derlenen iki sınıf dosyası farklı UID'ye sahip olabilir ve `readObject()` bir `InvalidClassException` fırlatır. Alanı elle `private static final long serialVersionUID` olarak sabitlemek, sınıfı geriye dönük uyumlu şekilde evrimleştirmenize (yeni alan eklemek gibi) izin verir; JVM yalnızca UID eşleşmesine bakar, alan sayısına bakmaz.

## Dikkat edilmesi gerekenler

- Serileştirilen bir nesnenin tüm alanları da `Serializable` olmalıdır; aksi halde `NotSerializableException` alırsınız. `transient` işaretlemek bu zorunluluktan muaf tutar.
- Kalıtımda üst sınıf `Serializable` değilse, onun alanları serileştirme/deserileştirme sırasında korunmaz; üst sınıfın no-arg constructor'ı çağrılarak varsayılan değerlere döner.
- `static` alanlar sınıfa, nesneye değil ait olduğu için hiçbir zaman serileştirmeye dahil edilmez.

## Denemek için

```
jshell> import java.io.*
jshell> record Point(int x, int y) implements Serializable {}
jshell> var baos = new ByteArrayOutputStream()
jshell> try (var o = new ObjectOutputStream(baos)) { o.writeObject(new Point(3, 4)); }
jshell> var in = new ObjectInputStream(new ByteArrayInputStream(baos.toByteArray()))
jshell> in.readObject()
```

Son satırda `Point[x=3, y=4]` değerinin geri geldiğini, `record`'ların da sadece `Serializable` işaretlenerek bu mekanizmayı kullanabildiğini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Serializable.html]
