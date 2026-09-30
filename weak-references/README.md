# Weak References

Bir nesneye normal (strong) bir referans olduğu sürece garbage collector o nesneyi asla toplamaz; bu genelde istenen davranıştır ama bir cache veya dinleyici (listener) kaydı gibi yapılarda tuzağa dönüşür: cache anahtarı olarak tutulan nesne artık başka hiçbir yerde kullanılmasa bile, sırf cache map'inde referansı olduğu için bellekte kalmaya devam eder. `WeakReference`, bir nesneye GC'nin görmezden geldiği türden bir referans tutarak bu sızıntıyı önler.

## GC'nin görmezden geldiği referans

```java
Object data = new byte[1_000_000];
WeakReference<Object> ref = new WeakReference<>(data);

data = null;
System.gc();

System.out.println(ref.get());
```

`ref` nesnesi `data`'ya işaret eder ama bu işaret GC açısından "yok sayılır"; yani `data = null` yapıldıktan sonra nesneye artık hiçbir strong referans kalmadığı için bir GC döngüsünde toplanabilir. `System.gc()` çağrısından sonra `ref.get()` genellikle `null` döner çünkü referans verilen nesne toplanmış, `WeakReference`'ın kendisi de içindeki bağı otomatik olarak temizlemiştir. `get()` her zaman `null` dönme garantisi taşımaz -- ne zaman toplanacağı JVM'in GC stratejisine bağlıdır -- ama nesnenin bellekte kalıcı olarak tutulmayacağı garantidir.

## WeakHashMap ile kendi kendini temizleyen cache

```java
Map<Key, Value> cache = new WeakHashMap<>();
cache.put(key, computeExpensive(key));
```

`WeakHashMap`, anahtarlarını `WeakReference` üzerinden tutar; `key` nesnesine map dışında başka strong referans kalmadığında GC o anahtarı toplar ve map'teki karşılık gelen giriş bir sonraki erişimde otomatik olarak silinir. Bu, `remove()` çağırmayı unutulan manuel cache temizliğine kıyasla bellek sızıntısını yapısal olarak engeller; ancak değer (`Value`) nesnesi anahtara güçlü referans tutuyorsa, anahtar zayıf da olsa değer üzerinden dolaylı bir döngü oluşup nesnenin toplanmasını engelleyebilir.

## Dikkat edilmesi gerekenler

- `WeakReference` senkron/deterministik bir temizlik mekanizması değildir; `get()` çağrısının ne zaman `null` döneceğini garanti edemezsiniz, bu yüzden kritik kaynak yönetimi (dosya, socket) için kullanılmaz.
- `SoftReference`, `WeakReference`'a benzer ama JVM bellek sıkışmadıkça nesneyi tutmaya çalışır; cache'ler için genelde `SoftReference` daha yumuşak bir seçenektir.
- `WeakHashMap` thread-safe değildir; eşzamanlı erişim gerekiyorsa `Collections.synchronizedMap` ile sarmalanmalıdır.

## Denemek için

```
jshell> var data = new Object()
jshell> var ref = new java.lang.ref.WeakReference<>(data)
jshell> data = null
jshell> System.gc()
jshell> ref.get()
```

`System.gc()` sonrası `ref.get()` çağrısının `null` döndüğünü gözlemleyin (garanti değil ama pratikte genellikle gerçekleşir).

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/WeakReference.html]
