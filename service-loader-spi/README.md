# service-loader-spi

Bir kütüphane yazarken, kullanıcı kodunun hangi somut sınıfı kullanacağını derleme zamanında bilmek istemezsiniz: arayüzü siz tanımlarsınız, implementasyonu başka bir modül sağlar. Bunu `new SomeImpl()` ile sabit kodlamak, kütüphanenizi tüketicinin sınıfına derleme zamanı bağımlılığıyla kilitler. `java.util.ServiceLoader`, bu bağımlılığı tersine çevirip implementasyonları çalışma zamanında, classpath (veya module path) üzerinden keşfederek yükler; bu desene `Service Provider Interface (SPI)` denir.

## Servis arayüzü ve sağlayıcı kaydı

```java
public interface GreetingProvider {
    String greet();
}

public class TurkishGreetingProvider implements GreetingProvider {
    public String greet() { return "Merhaba"; }
}
```

Sağlayıcıyı kaydetmek için `META-INF/services/` altında, arayüzün tam nitelikli adıyla aynı isimde bir dosya oluşturulur (`META-INF/services/com.example.GreetingProvider`), içeriği de implementasyonun tam nitelikli adıdır. `ServiceLoader`, bu metin dosyasını okuyarak hangi sınıfları örnekleyeceğini öğrenir; implementasyon sınıfının `public` ve parametresiz bir kurucuya sahip olması gerekir.

## Çalışma zamanında yükleme

```java
ServiceLoader<GreetingProvider> loader = ServiceLoader.load(GreetingProvider.class);
for (GreetingProvider provider : loader) {
    System.out.println(provider.greet());
}
```

`load()` çağrısı implementasyonları hemen örneklemez; dönen `ServiceLoader`, `Iterable` olarak lazy çalışır ve her `next()` çağrısında sırayla bir sağlayıcıyı bulur, sınıfını yükler ve örnekler. Birden fazla `jar` aynı arayüz için farklı `META-INF/services` girdileri sağlıyorsa, hepsi bu döngüde birer birer karşınıza çıkar; sıralama classpath sırasına bağlıdır ve garantili değildir.

## Ne zaman işe yarar

- Eklenti (`plugin`) mimarileri: çekirdek modül arayüzü tanımlar, her eklenti kendi `jar`'ında implementasyonunu kaydeder, çekirdek modül eklenti sınıflarını hiç tanımadan kullanır.
- JDBC sürücüleri (`java.sql.Driver`), `java.nio.file.spi.FileSystemProvider` gibi JDK'nin kendi SPI noktaları aynı mekanizmayı kullanır.
- Java 9+ modül sisteminde `module-info.java` içine `provides ArayuzAdi with ImplAdi;` ve `uses ArayuzAdi;` satırları eklenerek aynı keşif, dosya tabanlı `META-INF/services` yerine modül bildirimi üzerinden yapılabilir.

## Denemek için

```
jshell> interface Greeter { String hello(); }
jshell> class Tr implements Greeter { public String hello() { return "Selam"; } }
jshell> var loader = java.util.ServiceLoader.load(Greeter.class)
```

`jshell`'de `META-INF/services` dosyası olmadığı için döngü boş döner; gerçek bir sınama için implementasyonu ayrı bir `jar`'a koyup ilgili kayıt dosyasını eklemeniz, sonra bu `jar`'ı classpath'e eklemeniz gerekir. `ServiceLoader.load(...).findFirst()` ile tek bir sağlayıcı beklediğiniz durumlarda `Optional<T>` olarak sonucu alabilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ServiceLoader.html]
