# WatchService ile Dosya Sistemi Olayları

Bir dizindeki değişiklikleri yakalamanın saf yolu, dosyaları periyodik olarak `Files.list`/`Files.getLastModifiedTime` ile tarayıp önceki durumla karşılaştırmaktır (polling). Bu yaklaşım CPU'yu boşa yorar ve tarama aralığından kısa süren değişiklikleri kaçırabilir. `java.nio.file.WatchService`, işletim sisteminin kendi dosya sistemi bildirim mekanizmasına (inotify, FSEvents, ReadDirectoryChangesW) abone olarak değişiklikleri olay tabanlı, bekleyerek (`blocking`) teslim eder.

## Dizini izlemeye kaydolma

```java
import java.nio.file.*;

WatchService watcher = FileSystems.getDefault().newWatchService();
Path dir = Path.of("/tmp/izlenen");
dir.register(watcher,
        StandardWatchEventKinds.ENTRY_CREATE,
        StandardWatchEventKinds.ENTRY_MODIFY,
        StandardWatchEventKinds.ENTRY_DELETE);
```

`register` çağrısı `Path`'i belirtilen olay türleri için bir `WatchKey` üretecek şekilde `WatchService`'e bağlar; bu, bir alt kaynağa aboneliktir, dosya sistemine bir değişiklik yazmaz. Kayıt sadece doğrudan o dizinin içeriğini kapsar, alt dizinler (`recursive`) otomatik izlenmez; her alt dizin için ayrı `register` çağrısı gerekir.

## Olayları tüketme döngüsü

```java
while (true) {
    WatchKey key = watcher.take();  // olay gelene kadar bloklar
    for (WatchEvent<?> event : key.pollEvents()) {
        Path changed = (Path) event.context();
        System.out.println(event.kind() + ": " + changed);
    }
    boolean valid = key.reset();
    if (!valid) break;  // dizin silinmiş/erişilemez, anahtar artık geçersiz
}
```

`take()` bir olay gerçekleşene kadar çağıran thread'i bloklar; aynı anahtarın birikmiş olaylarını tek seferde almak için `pollEvents()` kullanılır. Kritik nokta `key.reset()`'tir: bu çağrı yapılmadan aynı `WatchKey` üzerinde yeni olaylar sıraya alınmaz, dolayısıyla döngüde unutmak izlemeyi sessizce durdurur.

## Dikkat edilmesi gerekenler

- `OVERFLOW` olayı, işletim sisteminin dahili olay kuyruğu dolup bazı değişikliklerin kaybolduğunu bildirir; bu durumda uygulamanın kendi tutarlılık kontrolü (tam tarama) yapması gerekir.
- `event.context()` sadece değişen dosyanın dizine göre *bağıl* adını verir, tam yolu elde etmek için `dir.resolve(changed)` gerekir.
- Çok hızlı ardışık yazmalar tek bir `ENTRY_MODIFY` olarak toplanabilir; olay sayısına güvenip "kaç kez değişti" saymak güvenilir değildir.

## Denemek için

```
jshell> import java.nio.file.*
jshell> var ws = FileSystems.getDefault().newWatchService()
jshell> Path.of("/tmp").register(ws, StandardWatchEventKinds.ENTRY_CREATE)
jshell> Files.writeString(Path.of("/tmp/deneme.txt"), "x")
jshell> ws.take().pollEvents()
```

Son satır, `/tmp` altında az önce oluşturulan dosyayı `ENTRY_CREATE` olayı olarak listeler.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/WatchService.html]
