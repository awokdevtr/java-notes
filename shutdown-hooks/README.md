# shutdown-hooks

Bir JVM süreci kapanırken (normal çıkış, `System.exit()`, veya `Ctrl+C` ile gönderilen `SIGTERM`) açık kaynakları kapatmak, bağlantı havuzlarını boşaltmak ya da son metrikleri göndermek isteyebilirsiniz. Ancak `main` metodu bittiğinde ya da süreç dış bir sinyalle sonlandırılmaya çalışıldığında bu temizlik kodunu nereye yazacağınız açık değildir. `Runtime.addShutdownHook()`, tam olarak bu kapanış anını yakalamak için JVM'e kayıtlı bir thread ekler.

## Temel kullanım

```java
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    System.out.println("kaynaklar kapatılıyor...");
}));

System.out.println("ana iş bitti");
```

Kayıt edilen her `Thread`, JVM kapanış sürecine girdiğinde `start()` edilir; JVM, tüm shutdown hook thread'leri bitene kadar süreci sonlandırmaz. Hook'lar birbirine göre `concurrent` çalışır, yani sıralama garantisi yoktur ve her hook kendi içinde thread-safe olmalıdır. Hook içinde yeni bir hook eklemeye veya mevcut birini kaldırmaya çalışmak `IllegalStateException` fırlatır, çünkü kapanış dizisi başladıktan sonra kayıt listesi dondurulur.

## Kaldırma ve zaman sınırı

```java
Thread hook = new Thread(() -> cleanup());
Runtime.getRuntime().addShutdownHook(hook);
// ... daha sonra, kapanış başlamadan önce:
Runtime.getRuntime().removeShutdownHook(hook);
```

`removeShutdownHook()` yalnızca kapanış dizisi henüz başlamamışken çalışır; aksi halde `IllegalStateException` alırsınız. Hook'lar, `System.exit()`'in çağrıldığı veya tüm `user thread`'lerin bittiği normal kapanışlarda çalışır, ama `Runtime.halt()` çağrısı veya JVM'in `kill -9` gibi zorla sonlandırılması durumunda hiç tetiklenmez; bu yüzden hook'u kritik veri bütünlüğü için tek güvence olarak kullanmamalısınız.

## Dikkat edilmesi gerekenler

- Hook içinde uzun süren işlemler süreci kapanışta bloke eder; JVM, hook'ların bitmesini sonsuza kadar bekler, bu yüzden hook kodunu hızlı ve zaman sınırlı tutmak gerekir.
- `daemon thread`'lerin aksine shutdown hook'lar JVM'in kapanmasını engellemez, sadece kapanış anında çalışır; ama hook içinde yeni bir `user thread` başlatmak kapanışı geciktirebilir.
- Birden fazla hook kaydedilebilir ve her biri kendi thread'inde paralel çalışır; ortak kaynağa erişen hook'lar arasında senkronizasyon sizin sorumluluğunuzdadır.

## Denemek için

```
jshell> Runtime.getRuntime().addShutdownHook(new Thread(() -> System.out.println("kapaniyorum")))
jshell> /exit
```

`jshell`den çıkarken konsola `kapaniyorum` yazdığını görürsünüz; bu, kayıtlı hook'un JVM kapanış sinyaliyle gerçekten tetiklendiğini doğrular.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Runtime.html#addShutdownHook(java.lang.Thread)]
