# StringBuilder vs StringBuffer

`String` değişmez olduğu için her birleştirme işlemi yeni bir nesne üretir. Bunu önlemek amacıyla standart kütüphane iki ayrı değişebilir (mutable) metin biriktirici sunar: `StringBuilder` ve `StringBuffer`. İkisi de aynı API'ye (`append`, `insert`, `delete`, `reverse`...) sahiptir, ama aralarındaki tek fark eşzamanlılık garantisidir — ve bu fark, hangisini seçeceğinizi doğrudan belirler.

## Senkronizasyon farkı

```java
StringBuffer buffer = new StringBuffer();
buffer.append("a").append("b"); // her metot synchronized

StringBuilder builder = new StringBuilder();
builder.append("a").append("b"); // senkronizasyon yok
```

`StringBuffer`'ın her metodu `synchronized` olarak işaretlenmiştir; yani her çağrı iç kilidi almak zorundadır, bu da çoklu thread tarafından paylaşılsa bile tek thread'li kullanımda bile gereksiz kilit maliyeti getirir. `StringBuilder` ise Java 5 ile eklendi ve hiçbir senkronizasyon yapmaz; tek thread içinde (ki çoğu kullanım senaryosu budur) belirgin ölçüde daha hızlıdır. İkisi de aynı dahili `char[]`/`byte[]` tamponunu ikiye katlayarak büyütme stratejisini kullanır, bu yüzden kapasite yönetimi davranışı aynıdır.

## Paylaşılan durumun tehlikesi

```java
StringBuilder shared = new StringBuilder();
Runnable task = () -> shared.append("x");
new Thread(task).start();
new Thread(task).start();
```

`StringBuilder` senkronize olmadığı için birden fazla thread aynı örneği eşzamanlı değiştirirse `append` çağrıları birbirine karışabilir, hatta iç tamponun yeniden boyutlandırılması sırasında tutarsız bir durum oluşabilir. `StringBuffer` tek başına bunu önler ama yalnızca tek bir `append` çağrısını atomik yapar; bir dizi `append`/`insert` işlemini bütün olarak korumak için yine de dışarıdan ek senkronizasyon gerekir, bu yüzden `StringBuffer`'ın "thread-safe" garantisi pratikte sınırlıdır.

## Ne zaman ise yarar

- Tek thread içindeki normal string biriktirme işlerinde (döngüde birleştirme, metot içi oluşturma) her zaman `StringBuilder` kullanın.
- `StringBuffer`, eski (legacy) API'lerle uyumluluk veya gerçekten tek bir örneğin birden fazla thread tarafından paylaşıldığı nadir durumlar dışında artık önerilmez; genelde daha iyi bir alternatif, biriktirmeyi thread-local yapıp sonucu paylaşmaktır.
- Performans farkı küçük dizilerde göz ardı edilebilir olsa da, büyük döngülerde `StringBuffer`'ın kilit maliyeti ölçülebilir hale gelir.

## Denemek için

```
jshell> import java.lang.reflect.Method
jshell> java.util.Arrays.stream(StringBuffer.class.getDeclaredMethods()).filter(m -> m.getName().equals("append") && java.lang.reflect.Modifier.isSynchronized(m.getModifiers())).count()
```

Bu, `StringBuffer.append` metotlarının `synchronized` olduğunu doğrudan reflection ile gösterir; aynı sorguyu `StringBuilder.class` ile çalıştırırsanız sonuç `0` çıkar.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuffer.html]
