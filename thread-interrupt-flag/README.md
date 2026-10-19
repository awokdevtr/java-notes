# thread-interrupt-flag

Çalışan bir thread'i dışarıdan durdurmak istediğinizde `Thread.stop()` kullanmak cazip görünür, ama bu metot kaynakları tutarsız bir durumda bırakabileceği için uzun süredir `deprecated`'tır. Java bunun yerine kooperatif bir mekanizma sunar: her thread'in içinde `interrupt status` adında bir boolean bayrak tutulur. `interrupt()` çağrısı bu bayrağı sadece `true` yapar; thread'i zorla durdurmaz, kontrolü hedef thread'in kendi koduna bırakır.

## Bayrağı ayarlama ve kontrol etme

```java
Thread worker = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // is yap
    }
    System.out.println("Dongu durdu, bayrak: " + Thread.currentThread().isInterrupted());
});
worker.start();
worker.interrupt();
```

`isInterrupted()` bir örnek metodudur, bayrağı okur ama sıfırlamaz. Döngü her turda bu bayrağı kontrol ederek `interrupt()` çağrısına kooperatif biçimde yanıt verir; thread kendi isteğiyle durur, dışarıdan zorla kesilmez.

## InterruptedException ve bayrağın temizlenmesi

```java
Thread worker = new Thread(() -> {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        System.out.println("Uyku kesildi, bayrak temizlendi: " + Thread.currentThread().isInterrupted());
        Thread.currentThread().interrupt(); // bayragi yeniden isaretle
    }
});
worker.start();
worker.interrupt();
```

`sleep`, `wait`, `join` gibi bloklayan metotlar interrupt edildiğinde `InterruptedException` fırlatır ve bu fırlatma sırasında bayrağı otomatik olarak temizler (`false` yapar). Bu yüzden `catch` bloğunda istisnayı sessizce yutmak, interrupt sinyalini tamamen kaybetmenize yol açar; çağıran kodun durumu görebilmesi için bayrağı `Thread.currentThread().interrupt()` ile yeniden işaretlemek yaygın bir kuraldır.

## Dikkat edilmesi gerekenler

- `Thread.interrupted()` (static) çağrılan thread'in bayrağını okur **ve sıfırlar**; `isInterrupted()` (instance) ise sadece okur, sıfırlamaz. İkisini birbirine karıştırmak kaybolan sinyallere yol açar.
- `InterruptedException`'ı yakalayıp hiçbir şey yapmadan geçmek (`catch (InterruptedException e) {}`) anti-pattern'dir; en azından bayrağı yeniden ayarlayın veya metodu `throws` ile yukarı iletin.
- Bir thread interrupt edildiğinde illa istisna fırlamaz; CPU-yoğun bir döngüde `InterruptedException` fırlatan bir çağrı yoksa bayrağı elle kontrol etmek sizin sorumluluğunuzdadır.

## Denemek için

```
jshell> Thread t = new Thread(() -> { try { Thread.sleep(10000); } catch (InterruptedException e) { System.out.println("kesildi"); } }); t.start(); t.interrupt();
```

Birkaç saniye içinde `"kesildi"` çıktısını görürsünüz; `sleep` beklemeyi tamamlamadan `InterruptedException` ile uyanır ve bayrak bu fırlatma sırasında otomatik temizlenir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html#interrupt()]
