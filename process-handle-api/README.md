# process-handle-api

`ProcessBuilder` ile yeni bir süreç başlatmak kolaydır, ama ne kendi başlattığınız süreci sonradan sorgulamak ne de o anki JVM'i ya da onun çocuklarını `Process` nesnesi olmadan incelemek mümkündür. Java 9 ile gelen `ProcessHandle`, işletim sisteminin herhangi bir sürecine -kendi başlatmadığınız biri dahil- PID üzerinden erişip canlı mı, kim onun ebeveyni, çocukları kimler sorularını platforma özgü kod yazmadan yanıtlamanızı sağlar.

## Güncel JVM'i ve çocuklarını incelemek

```java
ProcessHandle current = ProcessHandle.current();
System.out.println("PID: " + current.pid());

current.info().commandLine().ifPresent(System.out::println);
current.info().startInstant().ifPresent(System.out::println);

current.children().forEach(child ->
    System.out.println("child pid: " + child.pid()));
```

`ProcessHandle.current()` çalışan JVM'in kendi handle'ını döner; `info()` ise komut satırı, başlangıç zamanı, kullanıcı ve CPU süresi gibi verileri bir `ProcessHandle.Info` olarak paketler. Bu bilgilerin her biri işletim sistemine ve izinlere bağlı olarak eksik olabileceğinden `Optional` döner, bu yüzden doğrudan `get()` çağırmak yerine `ifPresent`/`orElse` kullanılmalıdır. `children()` sadece doğrudan alt süreçleri, `descendants()` ise tüm alt ağacı döner.

## PID ile keyfi bir süreci bulup sonlandırmak

```java
long pid = 12345L;
ProcessHandle.of(pid).ifPresentOrElse(handle -> {
    handle.onExit().thenRun(() -> System.out.println("süreç kapandı"));
    boolean requested = handle.destroy();
    System.out.println("sonlandırma istendi: " + requested);
}, () -> System.out.println("bu PID'de süreç yok"));
```

`ProcessHandle.of(pid)` o PID hâlâ canlıysa bir `Optional<ProcessHandle>` döner; süreç çoktan bitmişse veya hiç var olmamışsa boş döner, bu yüzden PID'leri asla "varlığının kanıtı" gibi saklamamak gerekir. `destroy()` yalnızca sonlandırma *isteğinde bulunur* ve bunun başarılı olup olmadığını garanti etmez; gerçek bitişi öğrenmenin güvenilir yolu `CompletableFuture` döndüren `onExit()`'tir. `destroyForcibly()` ise daha sert bir sinyal gönderir (Unix'te `SIGKILL` gibi).

## Ne zaman işe yarar

- Bir uygulamanın kendi başlattığı harici süreçlerin (örn. bir alt derleme aracı) hâlâ çalışıp çalışmadığını `Process` referansını elde tutmadan, sadece PID ile denetlemek.
- Sızıntı yapan zombi alt süreçleri bulup `descendants()` ile toplu olarak sonlandırmak.
- Platforma özgü `ps`/`tasklist` komutlarını `ProcessBuilder` ile parse etmek yerine taşınabilir bir API kullanmak.

## Denemek için

```
jshell> var self = ProcessHandle.current()
jshell> self.pid()
jshell> self.info().commandLine()
jshell> self.children().forEach(System.out::println)
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ProcessHandle.html]
