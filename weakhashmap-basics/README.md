# WeakHashMap

Normal bir `HashMap`, içine koyduğun her anahtarı `strong reference` ile tutar; bu referans harita yaşadığı sürece canlıdır ve anahtar nesnesi, haritadan açıkça silinmeden `garbage collector` tarafından toplanamaz. Bu, bir nesneye bağlı geçici meta veri ya da cache tutarken sızıntıya (`memory leak`) yol açabilir: anahtar olarak kullandığın nesne başka hiçbir yerden erişilemez hale gelse de harita onu diri tutmaya devam eder. `WeakHashMap`, anahtarlarını `WeakReference` üzerinden tutan bir `Map` uygulamasıdır; anahtar nesnesine başka hiçbir güçlü referans kalmadığında, `GC` o anahtarı toplayabilir ve ilgili girdi haritadan kendiliğinden düşer.

## Anahtar GC'ye takılınca girdi kaybolur

```java
Map<Object, String> cache = new WeakHashMap<>();

Object key = new Object();
cache.put(key, "metadata");
System.out.println(cache.size());

key = null;
System.gc();
Thread.sleep(100);
System.out.println(cache.size());
```

`key` değişkeni haritaya eklendiğinde boyut `1`'dir. `key = null` satırıyla nesneye giden tek `strong reference` koparılır; haritanın kendisi anahtarı sadece `weak reference` ile tuttuğu için nesne artık ulaşılamaz durumdadır. `System.gc()` çalıştığında `GC` bu anahtarı toplar ve `WeakHashMap` ilgili girdiyi arka planda temizler; bu yüzden ikinci `size()` çağrısı genelde `0` döner (kesin zamanlama `GC`'ye bağlıdır, garanti değildir).

## Ne zaman ise yarar

- Bir nesneye iliştirilmiş ek veriyi (`metadata`, `listener kaydı`, `cache girdisi`) tutarken, o nesnenin yaşam süresini haritanın uzatmasını istemediğin durumlarda.
- `ClassLoader` veya framework'lerin, uygulamanın canonicalization haritalarında (örneğin bir sınıfa bağlı ek bilgi tutarken) sınıfın `unload` edilmesini engellememek için.

## Dikkat edilmesi gerekenler

- Eşdeğer, bir anahtarın canlılığı sadece haritanın kendisine değil, dışarıdaki güçlü referanslara bağlıdır; aynı anahtarı başka bir yapıda (liste, başka harita) güçlü tutuyorsan girdi asla düşmez.
- Value'lar normal (`strong`) referanstır; value içinden anahtara geri referans varsa (örneğin anahtarı value içinde saklıyorsan) anahtarı yine diri tutup sızıntıyı geri getirirsin.
- Thread-safe değildir; eşzamanlı erişim için harici senkronizasyon veya `Collections.synchronizedMap` gerekir.
- Girdilerin ne zaman düştüğü `GC` zamanlamasına bağlıdır; "hemen silinir" varsayımıyla deterministik mantık kurulmamalıdır.

## Denemek için

```
jshell> var m = new java.util.WeakHashMap<Object, String>()
jshell> var k = new Object(); m.put(k, "v"); m.size()
jshell> k = null; System.gc(); Thread.sleep(200); m.size()
```

İlk `size()` çağrısı `1` döner; `k`'yi `null` yapıp `GC`'yi tetikledikten sonra ikinci `size()` çağrısı genellikle `0` döner, çünkü anahtara giden tek güçlü referans koptu.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/WeakHashMap.html]
