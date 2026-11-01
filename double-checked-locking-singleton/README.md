# Double-checked locking ile singleton

Bir singleton'ı lazy (ihtiyaç anında) oluşturmak isterseniz `getInstance()` metodunu tamamen `synchronized` yapmak akla gelir, ama bu her çağrıda kilit almayı zorunlu kılar ve nesne bir kez oluşturulduktan sonra bile gereksiz yere thread'leri sıraya sokar. Double-checked locking, kilidi sadece nesne henüz yokken alarak bu maliyeti ortadan kaldırmayı hedefler; ama `volatile` olmadan yanlış uygulanırsa yarı inşa edilmiş (partially constructed) bir nesne başka bir thread'e sızabilir.

## Yanlış ve doğru hali

```java
class Config {
    private static volatile Config instance;

    static Config getInstance() {
        if (instance == null) {            // 1. kontrol: kilitsiz
            synchronized (Config.class) {
                if (instance == null) {    // 2. kontrol: kilitli
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}
```

`instance` alanı `volatile` olmasaydı, JIT derleyici ve CPU `new Config()` içindeki adımları yeniden sıralayabilirdi: bellek ayrılır, `instance` referansı set edilir, constructor gövdesi *sonra* çalışır. Başka bir thread bu arada birinci kontrolde `instance != null` görüp henüz constructor'ı tamamlanmamış, alanları varsayılan değerlerde bir nesneyi geri alabilirdi. `volatile`, bu yeniden sıralamayı engelleyip yazmanın `happens-before` ilişkisiyle tam olarak görünmesini garanti eder. İkinci kontrol ise kilidi beklerken aynı anda birden fazla thread'in ayrı ayrı `new Config()` çağırmasını, yani birinci kontrolün kilitsiz olmasının getirdiği yarış penceresini kapatır.

## Daha basit alternatif: initialization-on-demand holder

```java
class Config {
    private static class Holder {
        static final Config INSTANCE = new Config();
    }

    static Config getInstance() {
        return Holder.INSTANCE;
    }
}
```

İç `Holder` sınıfı, `getInstance()` ilk çağrılana kadar JVM tarafından yüklenmez; sınıf yükleme/initialization adımları zaten JVM tarafından thread-safe garanti edilir, bu yüzden hiçbir kilide veya `volatile`'a gerek kalmaz. Çoğu durumda double-checked locking'den daha az hataya açık olduğu için tercih edilir; double-checked locking genelde eski kod tabanlarında veya kilidin manuel kontrolü gerektiği özel senaryolarda görülür.

## Dikkat edilmesi gerekenler

- `volatile` eklemeyi unutmak kod derlenir ve çoğu zaman çalışır görünür; hata sadece belirli JIT optimizasyonları ve çok çekirdekli yoğun eşzamanlılık altında ortaya çıkar, bu yüzden test etmesi zordur.
- İkinci kontrol olmadan sadece kilidi alıp kontrolü tekrarlamamak, nesnenin her thread tarafından ayrı ayrı yeniden oluşturulmasına yol açar.
- Enum singleton (`enum Config { INSTANCE }`) serialization ve reflection'a karşı da güvenli, JVM tarafından garanti edilen tek örneklilik sunduğu için genelde holder deseninden de daha sağlam kabul edilir.

## Denemek için

```
jshell> class Holder { static class H { static final String V = "hazır"; } static String get() { return Holder.H.V; } }
jshell> Holder.get()
jshell> class C { static volatile Object i; static Object get() { if (i == null) synchronized (C.class) { if (i == null) i = new Object(); } return i; } }
jshell> C.get() == C.get()
```

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.3.1.4]
