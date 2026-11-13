# classloader-basics

Bir Java programı çalıştığında sınıflar diskten tek seferde yüklenmez; JVM, her sınıfı ilk ihtiyaç duyulduğunda `ClassLoader` zinciri üzerinden "lazy" olarak yükler. Bu mekanizmayı anlamadan `NoClassDefFoundError`, `ClassNotFoundException` ya da aynı sınıfın iki farklı yükleyiciden gelip `instanceof` kontrolünde beklenmedik `false` döndürmesi gibi hatalar anlamsız görünür.

## Üç seviyeli hiyerarşi

```java
ClassLoader appLoader = ClassLoaderBasics.class.getClassLoader();
System.out.println(appLoader);                       // AppClassLoader
System.out.println(appLoader.getParent());            // PlatformClassLoader
System.out.println(appLoader.getParent().getParent()); // null (Bootstrap)
```

JVM, sınıfları üç katmanlı bir yükleyici hiyerarşisiyle yükler: en üstte yerel kodla (native) yazılmış ve Java'dan göründüğünde `null` olarak raporlanan `bootstrap` yükleyici (`java.lang.*` gibi çekirdek sınıfları yükler), onun altında `java.sql` gibi platform modüllerini yükleyen `PlatformClassLoader`, en altta ise classpath'teki kendi sınıflarınızı yükleyen `AppClassLoader` yer alır. Her yükleyicinin tek bir ebeveyni (`parent`) vardır ve bu referans zinciri `getParent()` ile yukarı doğru izlenebilir.

## Parent-delegation modeli

```java
public class CustomLoader extends ClassLoader {
    @Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        System.out.println("Yukleniyor: " + name);
        return super.loadClass(name, resolve); // once ebeveyne devreder
    }
}
```

`loadClass()`'ın varsayılan davranışı önce ebeveyn yükleyiciye sorup, o sınıfı bulamazsa kendisinin yüklemeyi denemesidir; buna `parent-delegation` denir ve amacı, kötü niyetli ya da yanlışlıkla tanımlanmış bir `java.lang.String` sınıfının çekirdek sınıfın yerine geçmesini (`class spoofing`) engellemektir. Bu yüzden `java.*` paketindeki bir sınıfı kendi yükleyicinizle override etmeye çalışmak `SecurityException` ile sonuçlanır. Delegasyon zinciri her zaman yukarıdan aşağıya değil, önce yukarı sorup başarısız olursa aşağı iner.

## Dikkat edilmesi gerekenler

- Aynı `.class` dosyası iki farklı `ClassLoader` tarafından yüklenirse JVM bunları tamamen farklı iki tip sayar; bu yüzden `instanceof` veya `ClassCastException` ile karşılaşırsanız önce yükleyicilerin aynı olup olmadığını kontrol edin.
- `Class.forName(name)` varsayılan olarak çağıran sınıfın yükleyicisini kullanır ve sınıfı hemen `initialize` eder (`static` bloklar çalışır); `ClassLoader.loadClass()` ise initialize etmeden sadece yükler.
- Uygulama sunucuları (Tomcat gibi) her `web app` için ayrı bir `ClassLoader` kullanarak aynı kütüphanenin farklı sürümlerinin çakışmadan bir arada yaşamasını sağlar.

## Denemek için

```
jshell> ClassLoader cl = String.class.getClassLoader()
jshell> System.out.println(cl)
jshell> ClassLoader appCl = Object.class.getClassLoader().getClass().getClassLoader()
```

`String.class.getClassLoader()`'ın `null` döndüğünü görürsünüz, çünkü `String` bootstrap yükleyici tarafından yüklenir ve Java tarafında bu yükleyicinin bir nesne temsili yoktur. Kendi yazdığınız bir sınıfın `getClassLoader()`'ını çağırırsanız `app` (veya `platform`) yükleyicisini görürsünüz.

[Kaynak: https://docs.oracle.com/javase/8/docs/technotes/guides/lang/cl-mt.html]
