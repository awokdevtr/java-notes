# ThreadLocal

`SimpleDateFormat` gibi thread-safe olmayan bir nesneyi birden fazla thread paylaştığında, ya her çağrıda yeni nesne yaratmak zorunda kalırsınız ya da senkronizasyonla performanstan feragat edersiniz. `ThreadLocal`, aynı değişkenin her thread için ayrı bir kopyasını tutarak bu paylaşım problemini ortadan kaldırır; ekstra parametre geçirmeden veya kilitlemeden her thread kendi izole değerini okur ve yazar.

## Thread başına izole değer

```java
class DateFormatter {
    private static final ThreadLocal<SimpleDateFormat> FORMAT =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

    static String format(Date date) {
        return FORMAT.get().format(date);
    }
}
```

`ThreadLocal.withInitial` her thread ilk kez `get()` çağırdığında lambda'yı çalıştırıp o thread'e özel bir `SimpleDateFormat` örneği üretir; sonraki çağrılarda aynı thread için saklanan örnek döner. `FORMAT` alanı `static` olsa da, JVM içeride her `Thread` nesnesine bağlı bir map tutar ve `get()`/`set()` çağrıları o map üzerinden çalışır; böylece iki thread aynı anda `format()` çağırsa bile birbirinin `SimpleDateFormat` nesnesine asla dokunmaz.

## Thread pool'da temizlik zorunluluğu

```java
executor.submit(() -> {
    try {
        FORMAT.get().format(new Date());
    } finally {
        FORMAT.remove();
    }
});
```

Thread pool'daki thread'ler görev bitince yok olmaz, havuza geri döner ve başka görevler için yeniden kullanılır; `ThreadLocal` üzerindeki değeri temizlemezseniz o değer thread'in map'inde kalıcı kalır ve hem gereksiz bellek tüketir hem de bir sonraki görev, önceki göreve ait "eski" değeri görebilir. `remove()` çağrısı, görev bittiğinde o thread'e özgü girdiyi map'ten siler; bu yüzden `ThreadLocal` kullanan kodun `finally` bloğunda `remove()` çağırması, özellikle thread pool ortamında bellek sızıntısını önlemek için şarttır.

## Dikkat edilmesi gerekenler

- `ThreadLocal` bir nesneyi paylaşmaz, çoğaltır; her thread kendi kopyası üzerinde çalışır, bu yüzden thread'ler arası veri paylaşımı veya senkronizasyon gerektiren durumlar için uygun değildir.
- Uzun ömürlü thread pool'larda `remove()` çağrılmazsa, `ThreadLocalMap` içindeki girdiler `Thread` nesnesi canlı kaldığı sürece serbest bırakılmaz; bu klasik bir bellek sızıntısı kaynağıdır.
- `InheritableThreadLocal`, bir thread yeni bir thread başlattığında değeri alt thread'e otomatik kopyalamak için kullanılır; sıradan `ThreadLocal` bunu yapmaz.

## Denemek için

```
jshell> ThreadLocal<Integer> id = ThreadLocal.withInitial(() -> 0)
jshell> Runnable r = () -> System.out.println(Thread.currentThread().getName() + ": " + id.get())
jshell> new Thread(() -> { id.set(42); r.run(); }).start()
jshell> r.run()
```

İki farklı thread'in `id.get()` çağrısı farklı değerler döndürür; ana thread hiç `set()` çağırmadığı için `0` başlangıç değerini görmeye devam eder.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ThreadLocal.html]
