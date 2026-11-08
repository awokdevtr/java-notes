# reflection-basics

Bazen bir sınıfın alanlarına veya metotlarına, bu tipi derleme zamanında hiç bilmeden erişmeniz gerekir: serialization kütüphaneleri, dependency injection framework'leri, test araçları hepsi bunu yapar. Java'nın `java.lang.reflect` paketi, çalışma zamanında bir `Class` nesnesi üzerinden sınıfın yapısını inceleme ve hatta `private` üyelere erişme olanağı verir; ama bu güç, derleme zamanı tip güvenliğinin bir kısmını çalışma zamanına taşır.

## Sınıf yapısını inceleme

```java
Class<?> clazz = String.class;
// ya da: Class.forName("java.lang.String")

for (Method m : clazz.getDeclaredMethods()) {
    System.out.println(m.getName() + " -> " + m.getReturnType());
}
```

`getDeclaredMethods()` sadece o sınıfta tanımlanan metotları döndürür, miras alınanları değil; üst sınıfları da almak için `getMethods()` kullanılır ama o da sadece `public` üyeleri getirir. `Class.forName` ise sınıf adını string olarak alır ve statik blokları tetikleyerek sınıfı yükler, bu yüzden yan etkilere dikkat etmek gerekir.

## private alana `setAccessible` ile erişim

```java
class Secret {
    private int value = 42;
}

Field field = Secret.class.getDeclaredField("value");
field.setAccessible(true);
int v = (int) field.get(new Secret());
```

`getDeclaredField` özel alanı bulur ama Java'nın erişim denetimi varsayılan olarak buna izin vermez; `setAccessible(true)` bu denetimi atlar. Java 9'dan itibaren modül sistemi (`module-info.java`) ile kapatılmış paketlerde bu çağrı `InaccessibleObjectException` fırlatabilir, dolayısıyla "her zaman çalışır" varsayımı güvenli değildir.

## Dikkat edilmesi gerekenler

- Reflection ile yapılan çağrılar JIT tarafından normal çağrılar kadar optimize edilemez, performans maliyeti vardır.
- `NoSuchFieldException`, `NoSuchMethodException`, `IllegalAccessException` gibi checked exception'ları yönetmek gerekir.
- Sınıfın iç yapısına doğrudan bağımlılık oluşturduğu için refactoring'i kırılgan hale getirir; mümkünse normal API veya `MethodHandle`/`VarHandle` gibi daha tip güvenli alternatifler tercih edilmelidir.

## Denemek için

```
jshell> class Secret { private String name = "gizli"; }
jshell> var f = Secret.class.getDeclaredField("name")
jshell> f.setAccessible(true)
jshell> f.get(new Secret())
```

Son satırda `setAccessible` çağrılmadan `get` denenirse `IllegalAccessException` fırlatıldığını, çağrıldıktan sonra ise `"gizli"` değerinin doğrudan okunduğunu görürsünüz.

[Kaynak: https://docs.oracle.com/javase/tutorial/reflect/index.html]
