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
