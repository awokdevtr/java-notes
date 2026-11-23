# uncaught-exception-handler

Bir `Thread` içinde yakalanmamış bir `RuntimeException` veya `Error` fırlatıldığında, o thread sessizce sonlanır ve hatayı yutan bir `catch` bloğu yoksa varsayılan davranış sadece `System.err`'e bir `stack trace` yazmaktır; `main` thread dışında çalışan bir işçi thread için bu, programın görünürde "takılıp kalması" ya da sonucun hiç gelmemesi anlamına gelir çünkü exception, o thread'i başlatan koda hiçbir şekilde fırlatılmaz. `Thread.UncaughtExceptionHandler`, bu kaçak hataları merkezi bir yerde yakalamanın standart yoludur.

## Tek bir thread için handler

```java
Thread worker = new Thread(() -> {
    throw new IllegalStateException("beklenmeyen durum");
});
worker.setUncaughtExceptionHandler((t, e) -> {
    System.err.println(t.getName() + " çöktü: " + e);
});
worker.start();
```

`setUncaughtExceptionHandler()`, ilgili thread başlamadan önce çağrılmalıdır; thread çalışırken fırlatılan ve hiçbir `catch` ile yakalanmayan her `Throwable`, JVM tarafından otomatik olarak bu `handler`'ın `uncaughtException(Thread, Throwable)` metoduna iletilir. Bu çağrı, thread'in kendi yığınının en üstünde, sonlanma anında gerçekleşir; `handler` içinde fırlatılan yeni bir hata JVM tarafından yutulur, yani handler kodu kendi içinde güvenli olmalıdır.

## Tüm thread'ler için varsayılan handler

```java
Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    logToMonitoring(t.getName(), e);
});
```

`setDefaultUncaughtExceptionHandler()` statik bir metottur ve belirli bir `handler` atanmamış tüm thread'ler için JVM genelinde geçerli olur. Öncelik sırası şöyledir: önce thread'in kendi `setUncaughtExceptionHandler()` ile atanmış `handler`'ı, yoksa thread'in ait olduğu `ThreadGroup`'un `uncaughtException` davranışı, o da yoksa `Thread.setDefaultUncaughtExceptionHandler()` ile atanan global `handler` çalışır.

## Ne zaman işe yarar

- `ExecutorService` ile gönderilen görevlerde `Runnable` içindeki hatalar `Future.get()` çağrılmadığı sürece hiç görünmez; thread pool'un `ThreadFactory`'sinde her thread'e `uncaughtExceptionHandler` atamak, bu sessiz kayıpları merkezi loglamaya bağlamanın pratik yoludur.
- `main` thread'deki yakalanmamış hatalar JVM'i zaten sonlandırıp `stack trace`'i yazdırır; bu mekanizma özellikle arka planda çalışan, kimsenin doğrudan beklemediği işçi thread'ler için değerlidir.
- `try/catch` ile yakalanabilecek bir hatayı bilerek bu mekanizmaya bırakmak yanlış tasarımdır; bu, beklenmeyen/programlama hatası niteliğindeki son çare yakalayıcıdır, normal akış kontrolü için kullanılmaz.

## Denemek için

```
jshell> Thread t = new Thread(() -> { throw new RuntimeException("test"); })
jshell> t.setUncaughtExceptionHandler((th, ex) -> System.out.println(th.getName() + " -> " + ex.getMessage()))
jshell> t.start()
```

Konsolda `Thread-0 -> test` çıktısını görürsünüz; bu, hiçbir `catch` bloğu yazmadığınız halde hatanın atanmış `handler` tarafından yakalandığını doğrular.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.UncaughtExceptionHandler.html]
