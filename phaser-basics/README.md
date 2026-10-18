# phaser-basics

`CyclicBarrier`, bariyerde buluşması gereken thread sayısını yapıcıda sabitler; çalışma zamanında yeni worker eklemek veya birini çıkarmak desteklenmez. Thread sayısı dinamik olarak değişen, aynı zamanda birbirini takip eden birden çok aşamadan (phase) geçmesi gereken işler için bu yetersizdir. `Phaser`, katılımcı sayısının çalışma zamanında artıp azalabildiği, çok aşamalı senkronizasyon noktaları sunan daha esnek bir araçtır.

## Temel kullanım

```java
Phaser phaser = new Phaser(3);

Runnable worker = () -> {
    System.out.println(Thread.currentThread().getName() + " asama 1'i tamamladi");
    phaser.arriveAndAwaitAdvance();

    System.out.println(Thread.currentThread().getName() + " asama 2'yi tamamladi");
    phaser.arriveAndAwaitAdvance();
};

for (int i = 0; i < 3; i++) {
    new Thread(worker).start();
}
```

`new Phaser(n)` ile `n` katılımcı kayıtlı başlar. `arriveAndAwaitAdvance()` çağıran thread kendi işini bitirdiğini bildirir ve tüm kayıtlı katılımcılar aynı aşamaya ulaşana kadar bloke olur; son katılımcı geldiğinde `phaser` bir sonraki aşamaya (`phase` numarası bir artarak) geçer ve herkes serbest kalır. `CyclicBarrier`'dan farklı olarak bu döngü, aynı nesne üzerinde sınırsız sayıda aşama için tekrarlanabilir.

## Dinamik katılımcı yönetimi

```java
Phaser phaser = new Phaser(1);

Runnable lateWorker = () -> {
    phaser.register();
    try {
        System.out.println("Gec katilan worker isini yapiyor");
    } finally {
        phaser.arriveAndDeregister();
    }
};

new Thread(lateWorker).start();
phaser.arriveAndAwaitAdvance();
```

`register()` çalışma zamanında bariyere yeni bir katılımcı ekler, `bulkRegister(n)` ise birden fazlasını birden ekler. Bir thread işi tamamen bittiğinde `arriveAndDeregister()` çağırarak hem geldiğini bildirir hem de kendini kayıttan düşürür; böylece sonraki aşamalar kalan katılımcı sayısına göre ilerler. Bu, worker havuzunun büyüklüğü iş sırasında değişen sistemlerde `CyclicBarrier` ile taklit edilemeyen bir esneklik sağlar.

## Dikkat edilmesi gerekenler

- Kayıtlı katılımcı sayısı sıfıra düşerse `phaser` `terminated` duruma geçer; bu durumda `arrive()` ve `awaitAdvance()` hemen döner ve senkronizasyon etkisini kaybeder.
- `arriveAndAwaitAdvance()` çağırmadan önce o thread kayıtlı olmalıdır; kayıtsız bir thread'in `arrive()` çağırması `IllegalStateException` fırlatır.
- Ağaç yapısında (`tiered`) alt phaser'lar oluşturularak çok sayıda thread tek bir merkezi phaser'ı tıkamadan hiyerarşik olarak senkronize edilebilir; bu, büyük thread sayılarında `CyclicBarrier`'a göre ölçeklenebilirlik avantajı sunar.

## Denemek için

```
jshell> import java.util.concurrent.Phaser
jshell> var phaser = new Phaser(1)
jshell> phaser.getPhase()
jshell> phaser.arriveAndAwaitAdvance(); phaser.getPhase()
```

Her `arriveAndAwaitAdvance()` çağrısı sonrası `getPhase()` değerinin bir arttığını, `getRegisteredParties()` ile de kayıtlı katılımcı sayısını gözlemleyebilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Phaser.html]
