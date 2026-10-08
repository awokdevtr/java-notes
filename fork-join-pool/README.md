# fork-join-pool

Büyük bir işi normal bir `ExecutorService` ile paralelleştirmek istediğinizde, her alt görev için ayrı ayrı `submit` edip sonuçları elle birleştirmek hem kalabalık hem de dengesiz iş yüküne karşı verimsizdir: bazı thread'ler boşta kalırken bazıları kuyrukta bekler. `ForkJoinPool`, böl-ve-fethet tarzı görevleri otomatik olarak alt görevlere ayırıp boştaki thread'lere dağıtan, work-stealing algoritmasıyla çalışan özel bir havuzdur.

## RecursiveTask ile böl-ve-fethet

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] data;
    private final int start, end;

    SumTask(int[] data, int start, int end) {
        this.data = data; this.start = start; this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= 1000) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += data[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(data, start, mid);
        SumTask right = new SumTask(data, mid, end);
        left.fork();
        return right.compute() + left.join();
    }
}

ForkJoinPool pool = new ForkJoinPool();
long total = pool.invoke(new SumTask(data, 0, data.length));
```

`compute()` metodu, görev yeterince küçükse (`threshold`) doğrudan hesaplar; değilse kendini ikiye bölüp bir yarısını `fork()` ile başka bir thread'e devreder, diğer yarısını kendi thread'inde işler ve `join()` ile sonucu bekler. `fork()`/`join()` çifti, alt görevi arka planda çalıştırırken çağıran thread'i de boşa düşürmez.

## Work-stealing ve ortak havuz

Her worker thread'in kendi görev kuyruğu vardır; bir thread kendi kuyruğunu bitirince, boşta kalmak yerine başka bir thread'in kuyruğunun kuyruk-sonundan görev "çalar" (`work-stealing`). Bu sayede dengesiz bölünmüş işlerde bile thread'ler verimli kullanılır. `ForkJoinPool.commonPool()` ile JVM genelinde paylaşılan varsayılan havuz alınabilir; `parallelStream()` ve `CompletableFuture`'ın varsayılan asenkron metotları da arka planda bu ortak havuzu kullanır.

## Dikkat edilmesi gerekenler

- `compute()` içinde I/O veya uzun süreli bloklayan çağrılar yapmak havuzdaki sınırlı thread sayısını tüketir; `ForkJoinPool` CPU-yoğun, bloklamayan işler için tasarlanmıştır.
- `fork()` ile `compute()`'u karıştırmak performans kaybına yol açar: genel kural, bir alt görevi `fork()` edip diğerini doğrudan `compute()` ile hesaplamaktır ("sağ görevi hesapla, sol görevi çaldır").
- `commonPool()` tüm `parallelStream()` çağrıları arasında paylaşıldığından, orada çalışan bir görev bloke olursa ilgisiz kodlardaki paralel stream'ler de yavaşlar.

## Denemek için

```
jshell> import java.util.concurrent.ForkJoinPool
jshell> var pool = new ForkJoinPool(4)
jshell> pool.submit(() -> java.util.stream.IntStream.rangeClosed(1, 1000).parallel().sum()).get()
```

`parallel()` çağrısının arka planda `ForkJoinPool`'un `invoke`/`fork`/`join` mekanizmasını kullandığını, sonucun aynı toplamı verdiğini görebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html]
