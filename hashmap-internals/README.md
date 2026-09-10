# HashMap iç yapısı

`HashMap`, anahtar-değer çiftlerini `O(1)` ortalama karmaşıklıkla saklamayı vaat eder ama bu performans sihir değildir: anahtarın `hashCode()` değeri bir dizi (bucket array) içindeki konuma çevrilir ve çarpışan anahtarlar aynı bucket içinde biriktirilir. Bu iç mekanizmayı bilmeden `HashMap` kullanmak, kötü `hashCode()` uygulamalarının veya yanlış boyutlandırmanın performansı `O(1)`'den `O(n)`'e düşürdüğü durumları anlamayı zorlaştırır.

## Bucket'a yerleşim

```java
Map<String, Integer> stok = new HashMap<>();
stok.put("elma", 12);
stok.put("armut", 7);
```

Her `put` çağrısında anahtarın `hashCode()` değeri alınır, üst bitleri alt bitlerle XOR'lanarak yayılır (`hash spreading`) ve dizinin boyutuna göre bucket indeksi hesaplanır. Aynı bucket'a düşen farklı anahtarlar önce bağlı liste (`linked list`) olarak tutulur; Java 8 ile birlikte bir bucket'taki eleman sayısı `TREEIFY_THRESHOLD` (8) değerini aşarsa liste kırmızı-siyah ağaca (`red-black tree`) dönüştürülür, böylece kötü niyetli veya çok çarpışan anahtarlarda arama süresi `O(n)` yerine `O(log n)`'e çekilir.

## Yeniden boyutlandırma (resize)

```java
Map<Integer, String> harita = new HashMap<>(16);
for (int i = 0; i < 13; i++) {
    harita.put(i, "deger" + i);
}
```

`HashMap`'in varsayılan `load factor` değeri `0.75`'tir; eleman sayısı `kapasite * load factor`'ü aştığında (16 kapasitede 12. eleman) dizi iki katına çıkarılır ve tüm anahtarlar yeniden dağıtılır (`rehash`). Bu işlem `O(n)` maliyetlidir, bu yüzden nihai eleman sayısı biliniyorsa `HashMap`'i baştan yeterli kapasiteyle oluşturmak gereksiz `resize` maliyetinden kaçındırır.

## Dikkat edilmesi gerekenler

- Anahtar olarak kullanılan sınıflar `equals()` ile tutarlı bir `hashCode()` uygulamalıdır; aksi halde aynı mantıksal anahtar farklı bucket'lara düşüp bulunamaz.
- `hashCode()` sabit bir değer döndürüyorsa (örneğin her zaman `1`) tüm elemanlar tek bucket'a yığılır ve harita fiilen bağlı listeye döner.
- `HashMap` senkronize değildir; eşzamanlı erişim gerekiyorsa `ConcurrentHashMap` tercih edilmelidir.

## Denemek için

```
jshell> var m = new java.util.HashMap<Integer,String>(16); for (int i = 0; i < 13; i++) m.put(i, "v"+i); System.out.println(m.size());
```

`13` eleman eklendikten sonra `size()` `13` döner; bu noktada arka planda en az bir `resize` gerçekleşmiş olur çünkü `12` eşiği aşılmıştır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html]
