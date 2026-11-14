# process-builder-basics

Bir Java programından işletim sisteminin başka bir programını (bir kabuk komutu, bir script, harici bir araç) çalıştırmak gerektiğinde eski `Runtime.exec()` metodu tek bir `String` komut satırını kendi başına, platforma özgü kurallarla parçalara ayırır ve bu ayrıştırma boşluk içeren argümanlarda ya da özel karakterlerde kolayca yanlış sonuç verir. `ProcessBuilder`, komutu ve argümanlarını ayrı ayrı `String` elemanları olarak alarak bu ayrıştırma belirsizliğini ortadan kaldırır ve çalışma dizini, ortam değişkenleri, girdi/çıktı yönlendirmesi gibi ayarları açık metotlarla sunar.

## Süreç başlatma ve çıktısını okuma

```java
ProcessBuilder builder = new ProcessBuilder("ls", "-la", "/tmp");
builder.redirectErrorStream(true);
Process process = builder.start();

try (var reader = process.inputReader()) {
    reader.lines().forEach(System.out::println);
}

int exitCode = process.waitFor();
System.out.println("Cikis kodu: " + exitCode);
```

Her argüman ayrı bir dizi elemanı olduğu için kabuk tarafından yeniden yorumlanmaz; bu yüzden argüman içinde boşluk veya tırnak işareti geçse de ek kaçış (`escaping`) gerekmez. `redirectErrorStream(true)` çağrısı, standart hata akışını standart çıktıya birleştirerek tek bir `InputStream` üzerinden okumayı sağlar. `waitFor()` çağıran thread'i, alt süreç sonlanana kadar bloke eder ve işletim sisteminin döndürdüğü çıkış kodunu verir.

## Çalışma dizini ve ortam değişkenleri

```java
ProcessBuilder builder = new ProcessBuilder("printenv", "JAVA_NOTES");
builder.directory(new java.io.File("/tmp"));
builder.environment().put("JAVA_NOTES", "merhaba");
builder.inheritIO();
Process process = builder.start();
process.waitFor();
```

`directory()` alt sürecin çalışma dizinini belirler; verilmezse JVM'in kendi çalışma dizini kullanılır. `environment()` metodu, ebeveyn JVM'in ortam değişkenlerinin bir kopyasını döndürür; buradaki değişiklikler sadece bu `ProcessBuilder` ile başlatılacak süreci etkiler, JVM'in kendi ortamını değiştirmez. `inheritIO()` ise alt sürecin standart giriş/çıkış/hata akışlarını doğrudan JVM'in kendi akışlarına bağlar; bu, çıktıyı programatik olarak işlemek istemediğiniz, sadece ekrana düşürmek istediğiniz durumlarda `InputStream` okuma kodunu gereksiz kılar.

## Dikkat edilmesi gerekenler

- `start()` çağrısı sonrası alt sürecin çıktı akışını okumazsanız ve işletim sisteminin dahili buffer'ı dolarsa, alt süreç çıktısını yazmak için bloke olabilir ve `waitFor()` sonsuza kadar bekleyebilir; bu yüzden çıktıyı `redirectErrorStream` veya `inheritIO` ile yönetmek önemlidir.
- `waitFor(long, TimeUnit)` aşırı yüklemesi, süreç belirtilen sürede bitmezse bloke olmadan `false` döndürür; uzun sürebilecek harici komutlarda zaman aşımı eklemek için bu kullanılmalıdır.
- Komut adı mutlak yol içermiyorsa işletim sisteminin `PATH` değişkenine göre aranır; bu davranış platformdan platforma değişebilir.

## Denemek için

```
jshell> var p = new ProcessBuilder("echo", "merhaba dunya").redirectErrorStream(true).start()
jshell> new String(p.getInputStream().readAllBytes())
jshell> p.waitFor()
```

`echo` komutuna tek bir argüman olarak verilen `"merhaba dunya"` dizesinin kabuk tarafından ayrıştırılmadan, boşluğuyla birlikte tek bir argüman halinde korunduğunu görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ProcessBuilder.html]
