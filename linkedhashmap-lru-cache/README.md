# linkedhashmap-lru-cache

`HashMap` giriş sırasını hiç garanti etmez; iterasyon sırası bucket dağılımına göre değişir. `LinkedHashMap` ise her giriş için bir çift bağlı liste düğümü tutarak ekleme sırasını (ya da isteğe bağlı olarak erişim sırasını) korur. Bu ek yapı, sınırlı boyutlu bir önbelleği en az kod ile, kendi eviction mantığınızı yazmadan kurmanızı sağlar.

## accessOrder ile en son kullanılanı öne alma

```java
Map<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
map.put(1, "a");
map.put(2, "b");
map.put(3, "c");
map.get(1);
System.out.println(map.keySet()); // [2, 3, 1]
```

Üç parametreli constructor'daki son argüman `accessOrder`'dır. `true` verildiğinde `get`/`put` ile dokunulan her giriş, dahili bağlı listenin sonuna taşınır; böylece en az kullanılan giriş her zaman listenin başında kalır. Varsayılan değer `false` olduğundan normal kullanımda `LinkedHashMap` sadece ekleme sırasını korur, erişim sırasını değil.

## removeEldestEntry ile otomatik boyut sınırlama

```java
Map<Integer, String> cache = new LinkedHashMap<>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry<Integer, String> eldest) {
        return size() > 3;
    }
};
cache.put(1, "a");
cache.put(2, "b");
cache.put(3, "c");
cache.put(4, "d");
System.out.println(cache.keySet()); // [2, 3, 4]
```

`removeEldestEntry`, her `put` çağrısından sonra `LinkedHashMap` tarafından otomatik çağrılan `protected` bir kanca metottur; `true` döndürdüğünde en eski giriş (listenin başındaki) otomatik silinir. `accessOrder=true` ile birleştirildiğinde bu, tam bir LRU (`least recently used`) önbellek anlamına gelir: dolu kapasitede yeni ekleme, en uzun süredir dokunulmamış girdiyi eler.

## Dikkat edilmesi gerekenler

- `removeEldestEntry` varsayılan olarak her zaman `false` döner; override etmezseniz `LinkedHashMap` sınırsız büyür.
- `accessOrder=true` iken sadece `get`/`put` sırayı günceller; `keySet()` veya `entrySet()` üzerinde salt okuma iterasyon sırayı değiştirmez.
- Çoklu thread'den erişimde `LinkedHashMap` da `HashMap` gibi thread-safe değildir; paylaşımlı bir LRU önbellek için harici senkronizasyon ya da `Collections.synchronizedMap` gerekir.

## Denemek için

```
jshell> var m = new java.util.LinkedHashMap<Integer,String>(16, 0.75f, true)
jshell> m.put(1,"a"); m.put(2,"b"); m.put(3,"c"); m.get(1); System.out.println(m.keySet())
```

Çıktıda `1` anahtarının listenin sonuna taşındığını, yani `[2, 3, 1]` sırasını göreceksiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashMap.html]
