# HashMap vs Hashtable

`Hashtable`, Java 1.0'dan kalan ve `HashMap`'ten önce var olan bir `Map` implementasyonudur. İkisi de anahtar-değer çifti tutar ve benzer bir API sunar, ama aralarındaki fark sadece isimlendirme değil: thread güvenliği, `null` desteği ve performans konusunda birbirinden tamamen ayrışırlar. Eski kodu okurken veya legacy bir API ile çalışırken bu farkları bilmemek, beklenmedik `NullPointerException`'lara veya gereksiz kilitlenmelere yol açabilir.

## Senkronizasyon davranışı

```java
Hashtable<String, Integer> eski = new Hashtable<>();
HashMap<String, Integer> yeni = new HashMap<>();

eski.put("istek", 1);   // synchronized metot, her çağrıda kilit alınır
yeni.put("istek", 1);   // kilitsiz, thread-safe DEĞİL
```

`Hashtable`'ın her metodu `synchronized` olarak işaretlenmiştir; bu da her `get`/`put` çağrısının nesne seviyesinde tek bir kilit almasını, yani eşzamanlı erişimde tüm thread'lerin sırayla beklemesini gerektirir. `HashMap` hiçbir senkronizasyon yapmaz ve birden fazla thread tarafından eşzamanlı değiştirildiğinde veri bozulmasına veya sonsuz döngüye yol açabilir. Bu yüzden `Hashtable`'ın "thread-safe" olması bir avantaj gibi görünse de, kaba taneli kilitleme yüzünden pratikte `ConcurrentHashMap`'ten çok daha yavaştır.

## null anahtar ve değer desteği

```java
yeni.put(null, 1);        // çalışır, anahtar null olabilir
yeni.put("x", null);      // çalışır, değer null olabilir

eski.put(null, 1);        // NullPointerException fırlatır
```

`HashMap` tek bir `null` anahtara ve istenildiği kadar `null` değere izin verir. `Hashtable` ise ikisine de izin vermez ve `NullPointerException` fırlatır; bu davranış, `Hashtable`'ın iç yapısının `null`'u "anahtar bulunamadı" durumuyla ayırt edemediği eski bir tasarım kısıtından kaynaklanır.

## Dikkat edilmesi gerekenler

- `Hashtable` *legacy* bir sınıftır; yeni kodda tercih edilmemeli. Thread güvenliği gerekiyorsa `ConcurrentHashMap`, gerekmiyorsa düz `HashMap` kullanılmalıdır.
- `Hashtable`'ın iteratörü `fail-fast` değildir, kendi `Enumeration` arayüzünü miras taşır; `HashMap`'in iteratörü ise değişiklik sırasında `ConcurrentModificationException` fırlatarak hatayı erken yakalar.
- `Vector` ile `ArrayList` arasındaki ilişki de aynı mantığı izler: `Vector` eski ve senkronize, `ArrayList` modern ve senkronize değildir.

## Denemek için

```
jshell> var h = new java.util.Hashtable<String,Integer>(); h.put(null, 1)
jshell> var m = new java.util.HashMap<String,Integer>(); m.put(null, 1); System.out.println(m)
```

İlk satır `NullPointerException` fırlatır, ikinci satır `{null=1}` çıktısını verir — aynı görünen iki `Map` implementasyonunun `null` karşısında nasıl ayrıştığını doğrudan gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Hashtable.html]
