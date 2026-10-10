# transient-keyword

Bir sınıf `Serializable` arayüzünü uyguladığında, varsayılan olarak her alan serileştirme sırasında bayt akışına yazılır. Ancak şifre, önbelleğe alınmış hesaplama sonucu ya da `Thread`/`Socket` gibi serileştirilemeyen bir kaynak referansı taşıyan alanlar bu akışa dahil edilmemelidir. `transient` anahtar kelimesi, bir alanı nesnenin kalıcı durumundan hariç tutarak bu sorunu çözer.

## Temel kullanım

```java
class Session implements Serializable {
    private final String username;
    private transient String authToken;

    Session(String username, String authToken) {
        this.username = username;
        this.authToken = authToken;
    }
}
```

`authToken` alanı `transient` olarak işaretlendiği için `ObjectOutputStream` bu alanı yazmaz. Nesne tekrar `ObjectInputStream` ile okunduğunda `authToken`, türünün varsayılan değerine döner: referans tipler için `null`, sayısal tipler için `0`, `boolean` için `false`. `final` bir alan bile `transient` olabilir; derleyici bunu constructor atamasıyla değil deserileştirme mekanizmasıyla uyumlu sayar çünkü deserileştirme constructor'ı hiç çağırmaz.

## Deserileştirme sonrası alanı yeniden kurmak

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject();
    this.authToken = TokenStore.reissueFor(username);
}
```

Sınıf `private void readObject(ObjectInputStream)` metodunu tanımlarsa, JVM varsayılan mekanizma yerine bu metodu çağırır. `defaultReadObject()` çağrısı `transient` olmayan alanları normal şekilde doldurur; ardından `transient` alan elle, örneğin bir servis çağrısıyla yeniden hesaplanarak atanabilir. Bu desen, geçici durumu serileştirmeden saklamanın ve nesneyi tutarlı biçimde yeniden canlandırmanın standart yoludur.

## Dikkat edilmesi gerekenler

- `transient`, yalnızca Java'nın yerleşik serileştirme mekanizmasını etkiler; `static` alanlar zaten nesne durumuna ait olmadığından serileştirmeye hiç girmez, `transient` ile işaretlemeye gerek yoktur.
- Bir alanı `transient` yapmak, o alanın `equals()`/`hashCode()` veya normal metot çağrılarındaki davranışını değiştirmez; yalnızca `ObjectOutputStream`/`ObjectInputStream` akışını etkiler.
- JSON tabanlı serileştirme kütüphaneleri (Jackson, Gson gibi) `transient`'ı genellikle kendi kurallarına göre yorumlar; bazıları yok sayar, bazıları `java.io.Serializable` sözleşmesine saygı gösterir, bu yüzden varsayım yapmadan kütüphane dokümantasyonu kontrol edilmelidir.

## Denemek için

```
jshell> import java.io.*
jshell> class P implements Serializable { String name = "ali"; transient int cache = 42; }
jshell> var bos = new ByteArrayOutputStream(); new ObjectOutputStream(bos).writeObject(new P())
jshell> var p2 = (P) new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray())).readObject(); System.out.println(p2.name + " " + p2.cache)
```

Çıktıda `name` alanının `"ali"` olarak korunduğunu, ama `transient` olan `cache` alanının `42` yerine `0` olarak geri geldiğini görürsünüz.

[Kaynak: https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serial-arch.html]
