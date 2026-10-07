# java-notes

Java ile ilgili kısa, dar kapsamlı notlar. Her alt klasör tek bir konuyu kapsar: kısa bir
Türkçe açıklama, bir kod örneği ve resmi Java SE dokümantasyonuna (docs.oracle.com) bir kaynak
linki. Günde 2 kez otomatik olarak yeni bir konu eklenir.

## Konular

- [try-with-resources](try-with-resources/README.md) — kaynakların otomatik kapatılması ve bastırılmış istisnalar
- [record-classes](record-classes/README.md) — değişmez veri taşıyıcıları ve compact constructor
- [sealed-classes](sealed-classes/README.md) — kısıtlı hiyerarşiler ve exhaustive switch
- [hashmap-internals](hashmap-internals/README.md) — bucket yapısı, çarpışma çözümü ve resize mekanizması
- [comparable-vs-comparator](comparable-vs-comparator/README.md) — doğal sıra ile dışsal sıralama stratejisi arasındaki fark
- [optional-usage](optional-usage/README.md) — null kontrolünü tip sistemine taşıma ve zincirleme dönüşümler
- [string-pool-intern](string-pool-intern/README.md) — string literal paylaşımı ve intern() ile havuza katılma
- [text-blocks](text-blocks/README.md) — çok satırlı string literalleri ve otomatik girinti temizliği
- [concurrent-modification-exception](concurrent-modification-exception/README.md) — fail-fast iterator davranışı ve güvenli eleman kaldırma yöntemleri
- [arraylist-vs-linkedlist](arraylist-vs-linkedlist/README.md) — iç veri yapısı farkı ve erişim/ekleme maliyetlerinin karşılaştırması
- [checked-vs-unchecked-exception](checked-vs-unchecked-exception/README.md) — derleyicinin zorladığı sözleşme ile programcı hatasının işareti arasındaki fark
- [equals-hashcode-contract](equals-hashcode-contract/README.md) — koleksiyonlarda doğru arama için gereken tutarlılık sözleşmesi
- [priority-queue](priority-queue/README.md) — binary heap tabanlı öncelik sırası ve özel Comparator ile sıralama
- [switch-expression](switch-expression/README.md) — `->` sözdizimiyle fall-through olmadan değer döndüren switch ve `yield`
- [integer-cache-pitfall](integer-cache-pitfall/README.md) — autoboxing sırasında -128..127 aralığında paylaşılan Integer nesneleri ve `==` tuzağı
- [static-initialization-order](static-initialization-order/README.md) — sınıf yüklenirken statik alan ve blokların çalışma sırası, miras hiyerarşisindeki etkisi
- [instanceof-pattern-matching](instanceof-pattern-matching/README.md) — tip kontrolü ile cast'i tek adımda birleştiren pattern variable ve flow scoping
- [generic-wildcards](generic-wildcards/README.md) — `? extends`/`? super` ile producer/consumer ayrımı ve PECS ilkesi
- [finally-return-interaction](finally-return-interaction/README.md) — `finally` içindeki `return`'ün `try`'daki değeri ve istisnayı nasıl sessizce iptal ettiği
- [stream-api-operations](stream-api-operations/README.md) — ara (intermediate) ve terminal operasyonlar arasındaki fark, tembel değerlendirme ve tek kullanımlık stream kuralı
- [completablefuture-basics](completablefuture-basics/README.md) — bloklamayan asenkron görev zincirleme, sonuç dönüştürme ve exceptionally/handle ile hata yönetimi
- [enum-method-override](enum-method-override/README.md) — sabite özgü metot gövdeleri ile enum sabitlerinin farklı davranış tanımlaması
- [stringbuilder-vs-string-concatenation](stringbuilder-vs-string-concatenation/README.md) — `+` ile döngüde string birleştirmenin ara nesne maliyeti ve `StringBuilder` ile tek buffer üzerinde biriktirme
- [constructor-chaining](constructor-chaining/README.md) — `this()` ve `super()` ile constructor'lar arası zincirleme ve çalışma sırası
- [varargs](varargs/README.md) — `...` ile değişken sayıda argüman kabul eden metotlar ve overload çözümlemesiyle etkileşimi
- [collectors-groupingby](collectors-groupingby/README.md) — `Collectors.groupingBy` ile stream elemanlarını anahtara göre gruplama ve downstream collector ile özetleme
- [java-time-api](java-time-api/README.md) — `LocalDate`/`LocalDateTime`/`Instant` ile immutable tarih-zaman modeli, `Period` ve `Duration` ile aralık hesabı
- [functional-interface-method-reference](functional-interface-method-reference/README.md) — tek soyut metotlu arayüzler, lambda ataması ve dört method reference türü arasındaki fark
- [deque-usage](deque-usage/README.md) — `ArrayDeque` ile aynı yapıyı hem yığın (LIFO) hem kuyruk (FIFO) olarak kullanma
- [immutability-and-final](immutability-and-final/README.md) — `final` ile alan bağlama ve savunmacı kopyalamayla gerçek değişmezlik arasındaki fark
- [autoboxing-pitfalls](autoboxing-pitfalls/README.md) — `null` sarmalayıcıların örtük unboxing sırasında fırlattığı `NullPointerException` ve ternary operatöründeki gizli tip dönüşümü
- [var-type-inference](var-type-inference/README.md) — `var` ile derleme zamanında somut tipe bağlanan yerel değişken tip çıkarımı ve çıkarımın imkansız olduğu durumlar
- [volatile-keyword](volatile-keyword/README.md) — thread'ler arası görünürlük garantisi ve `volatile`'ın atomiklik sağlamamasının yol açtığı tuzak
- [overload-resolution](overload-resolution/README.md) — üç aşamalı metot seçim algoritması ve autoboxing ile varargs'ın çözümleme sırasına etkisi
- [synchronized-keyword](synchronized-keyword/README.md) — metot ve blok senkronizasyonuyla karşılıklı dışlama, static kilit ile örnek kilidi arasındaki fark
- [static-method-hiding](static-method-hiding/README.md) — static metotların derleme zamanı tipine göre çözülmesi ve override ile hiding arasındaki fark
- [array-covariance](array-covariance/README.md) — dizilerin kovaryant atanabilmesi ve buna karşılık generics'in invariant olmasının yol açtığı `ArrayStoreException`
- [linkedhashmap-lru-cache](linkedhashmap-lru-cache/README.md) — `accessOrder` ile erişim sırasını izleme ve `removeEldestEntry` ile otomatik boyut sınırlı LRU önbellek kurma
- [thread-local](thread-local/README.md) — her thread'e özel izole değer tutma ve thread pool'larda `remove()` ile bellek sızıntısını önleme
- [generic-type-erasure](generic-type-erasure/README.md) — generic tip parametrelerinin derleme sonrası silinmesi ve bunun yol açtığı heap pollution riski
- [bigdecimal-vs-double](bigdecimal-vs-double/README.md) — kayan nokta yuvarlama hatasına karşı ölçekli tamsayı temsili ve para hesaplamalarında yuvarlama stratejisi seçimi
- [multi-catch-exceptions](multi-catch-exceptions/README.md) — `catch (A | B e)` ile birbiriyle ilişkisiz istisna türlerini tek blokta yakalama ve alt sınıf birleşiminin derleme hatası olması
- [atomic-integer-cas](atomic-integer-cas/README.md) — `AtomicInteger` ile compare-and-swap tabanlı kilitsiz sayaç ve `compareAndSet` ile optimistic locking
- [weak-references](weak-references/README.md) — `WeakReference` ile GC'nin görmezden geldiği referans türü ve `WeakHashMap` ile kendi kendini temizleyen cache
- [interface-default-methods](interface-default-methods/README.md) — `default` metotlarla geriye dönük uyumlu arayüz genişletme ve elmas probleminin `Interface.super` ile çözümü
- [clone-method-pitfalls](clone-method-pitfalls/README.md) — `Object.clone()`'un varsayılan shallow copy davranışı, `Cloneable` sözleşmesinin tuhaflıkları ve deep copy alternatifleri
- [arrays-aslist-pitfall](arrays-aslist-pitfall/README.md) — `Arrays.asList()`'in sabit boyutlu, diziyle paylaşılan bellekli liste görünümü ve bundan doğan `UnsupportedOperationException` tuzağı
- [record-patterns](record-patterns/README.md) — `instanceof` ve `switch` içinde `record`'ları tek adımda parçalarına ayırma ve iç içe geçmiş desenlerle zincirleme erişimi ortadan kaldırma
- [reentrantlock-vs-synchronized](reentrantlock-vs-synchronized/README.md) — `synchronized`'ın monitor tabanlı basit kilitlemesi ile `ReentrantLock`'un zaman aşımlı, kesilebilir ve adil kilitleme esnekliği arasındaki fark
- [copy-on-write-arraylist](copy-on-write-arraylist/README.md) — her yazmada diziyi kopyalayan `CopyOnWriteArrayList` ile kilitsiz okuma ve weakly consistent iterator semantiği
- [enummap-enumset](enummap-enumset/README.md) — enum sabitlerini `ordinal()` ile indeksleyen dizi tabanlı `EnumMap` ve bit vektörü tabanlı `EnumSet` ile `HashMap`/`HashSet`'e göre performans kazancı
- [navigablemap-navigation](navigablemap-navigation/README.md) — `TreeMap`'te `floor`/`ceiling`/`higher`/`lower` ile sıralı anahtar kümesinde en yakın eşleşmeyi `O(log n)`'de bulma
- [switch-pattern-guards](switch-pattern-guards/README.md) — `when` anahtar kelimesiyle pattern'e ek koşul bağlama ve bunun `exhaustiveness` denetimine etkisi
- [executor-service-thread-pool](executor-service-thread-pool/README.md) — `ExecutorService` ile thread yaratma maliyetini havuzlama, `Future` üzerinden sonuç alma ve `shutdown`/`shutdownNow` kapatma farkı
- [wait-notify-mechanics](wait-notify-mechanics/README.md) — `Object.wait`/`notify`/`notifyAll` ile monitor üzerinde thread koordinasyonu, `while` ile koşul kontrolü ve kaybolan bildirim tuzağı
- [instance-initializer-block](instance-initializer-block/README.md) — `static` olmayan `{}` bloğuyla constructor'lar arası ortak kurulum kodu ve anonim sınıflarda "double brace initialization" kalıbı
- [inner-vs-static-nested-class](inner-vs-static-nested-class/README.md) — `static` iç sınıfların dış örnekten bağımsızlığı ile inner class'ların dış nesneye tuttuğu gizli referansın yol açtığı bellek sızıntısı riski
- [covariant-return-types](covariant-return-types/README.md) — override edilen metotların üst sınıfın dönüş tipinin bir alt tipini döndürebilmesi ve `clone()` gibi kullanımlarda cast ihtiyacını ortadan kaldırması
- [countdown-latch](countdown-latch/README.md) — `CountDownLatch` ile birden fazla thread'in belirli sayıda işin tamamlanmasını beklemesi ve sayacın tek kullanımlık, sıfırlanamaz yapısı
