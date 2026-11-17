# enum-singleton-pattern

Klasik singleton deseni (`private` kurucu + statik `getInstance()`) seri hale getirmede (`serialization`) ve reflection karşısında kırılgandır: `readObject` yeni bir nesne yaratabilir, `setAccessible(true)` ile `private` kurucuya erişilip ikinci bir örnek üretilebilir. `enum` ile singleton tanımlamak bu sorunları dilin kendi garantileriyle ortadan kaldırır.

## Temel kullanım

```java
public enum ConfigRegistry {
    INSTANCE;

    private final Map<String, String> settings = new ConcurrentHashMap<>();

    public void set(String key, String value) {
        settings.put(key, value);
    }

    public String get(String key) {
        return settings.get(key);
    }
}
```

`ConfigRegistry.INSTANCE` tüm JVM boyunca tek bir nesnedir ve çağıranlar `ConfigRegistry.INSTANCE.set(...)` şeklinde erişir. JVM, her `enum` sabitinin sınıf yüklenirken tam olarak bir kez oluşturulmasını garanti eder; bu oluşturma `class loading` mekanizmasının kendisi tarafından thread-safe şekilde yapılır, ayrıca `synchronized` bloğuna gerek kalmaz.

## Serialization ve reflection'a karşı dayanıklılık

```java
ConfigRegistry a = ConfigRegistry.INSTANCE;
ConfigRegistry b = deserialize(serialize(a));
System.out.println(a == b); // true
```

Normal bir sınıfta `Serializable` uygulandığında varsayılan `readObject` davranışı yeni bir nesne üretir ve singleton'u bozar; bunu önlemek için elle `readResolve()` yazmak gerekir. `enum` türleri ise JVM tarafından özel olarak serileştirilir: akışa sadece sabitin adı yazılır, geri okurken `valueOf()` ile aynı sabit döndürülür. Aynı şekilde `Constructor.newInstance()` ile bir `enum` kurucusunu reflection üzerinden çağırmak `IllegalArgumentException` fırlatır, bu yüzden ikinci bir örnek reflection'la da üretilemez.

## Ne zaman işe yarar

- Uygulama ömrü boyunca tek örneği garanti altına almak istediğiniz durumsuz ya da paylaşımlı durumlu (`cache`, `registry`, `connection pool` yöneticisi gibi) bileşenlerde.
- Lazy başlatma gerekmiyorsa veya sınıf yüklenirken başlatmanın maliyeti kabul edilebilirse; `enum` sabitleri sınıf ilk kullanıldığında başlatılır, bu da zaten yeterince "lazy"dir.
- Kalıtım gerektiren singleton'larda uygun değildir, çünkü `enum` başka bir sınıftan türeyemez.

## Denemek için

```
jshell> enum Counter { INSTANCE; private int n; void inc() { n++; } int get() { return n; } }
jshell> Counter.INSTANCE.inc(); Counter.INSTANCE.inc(); Counter.INSTANCE.get()
```

İki ayrı `inc()` çağrısının aynı `INSTANCE` üzerinde biriktiğini, sonucun `2` döndüğünü görürsünüz; `new Counter()` yazmayı denerseniz derleme hatası alırsınız çünkü `enum` kurucuları dışarıdan çağrılamaz.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9]
