# volatile anahtar kelimesi

Birden fazla thread aynı alanı okuyup yazdığında, JIT derleyici ve CPU önbelleği yüzünden bir thread'in yazdığı yeni değer diğer thread'e hiç görünmeyebilir; her thread kendi CPU çekirdeğinin önbelleğinde eski değeri tutmaya devam edebilir. `volatile`, bir alan için bu görünürlük (visibility) sorununu derleyiciye ve JVM'e karşı garanti altına alan bir değiştiricidir.

## Görünürlük garantisi

```java
class Worker extends Thread {
    private volatile boolean running = true;

    void stopWorker() {
        running = false;
    }

    @Override
    public void run() {
        while (running) {
            // iş yap
        }
        System.out.println("durdu");
    }
}
```

`running` alanı `volatile` olmadan tanımlansaydı, JIT derleyici döngü içinde alanı bir kez register'a okuyup bir daha bellekten kontrol etmeyebilir, bu da `stopWorker()` başka bir thread'den çağrıldığında döngünün asla bitmemesine yol açabilir. `volatile` işaretlemesi her okumanın doğrudan ana bellekten (main memory) yapılmasını, her yazmanın da hemen ana belleğe yansıtılmasını zorunlu kılar; böylece bir thread'in yazdığı değeri diğer thread bir sonraki okumasında görür.

## Atomiklik sağlamaz

```java
class Counter {
    private volatile int count = 0;

    void increment() {
        count++; // read-modify-write, tek adım DEĞİL
    }
}
```

`volatile` yalnızca görünürlüğü garanti eder, `count++` gibi okuma-değiştirme-yazma işlemlerini atomik yapmaz. İki thread aynı anda `increment()` çağırdığında ikisi de aynı eski değeri okuyup üstüne bir ekleyebilir, sonuçta bir artış kaybolur. Sayaç gibi bileşik işlemler için `synchronized` ya da `java.util.concurrent.atomic.AtomicInteger` kullanılmalıdır.

## Ne zaman işe yarar

- Bir thread'in yazdığı, diğerlerinin sadece okuduğu bayrak (flag) veya durum alanlarında (`running`, `initialized` gibi) `volatile` yeterli ve `synchronized`'dan daha ucuzdur.
- Alan üzerinde `count++`, `if (x == null) x = new X()` gibi bileşik/koşullu işlemler varsa `volatile` tek başına yetersizdir; kilitleme veya atomik sınıflar gerekir.
- `volatile`, o alana yapılan yazmadan ÖNCEKİ diğer tüm alanlara yapılan yazmaların da görünür olmasını sağlar (happens-before ilişkisi); bu yüzden genelde bir nesnenin "hazır" olduğunu işaretleyen son alan olarak kullanılır.

## Denemek için

```
jshell> class Flag { volatile boolean on = true; }
jshell> Flag f = new Flag()
jshell> Thread t = new Thread(() -> { while (f.on) {} System.out.println("bitti"); })
jshell> t.start()
jshell> f.on = false
```

`volatile` olmadan aynı denemeyi tekrarlarsanız (alanı `boolean` yapıp `volatile`'ı kaldırarak), JIT optimizasyonu devreye girdiğinde thread'in hiç bitmeme ihtimali artar; tek bir jshell çalıştırmasında gözlemlemek garanti olmasa da, gerçek uygulamalarda bu fark ciddi hata kaynağıdır.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.3.1.4]
