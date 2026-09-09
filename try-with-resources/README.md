# try-with-resources

`InputStream`, `Connection`, `Scanner` gibi harici kaynaklarla çalışırken kaynağı elle kapatmayı unutmak, sızıntıya (`resource leak`) yol açan en yaygın hatalardan biridir. Klasik `finally` bloğuyla kapatma kodu hem kalabalık hem de kendi içinde hataya açıktır: `close()` çağrısı `NullPointerException` fırlatabilir ya da orijinal istisnayı gizleyebilir. `try-with-resources`, bu kapatma işini dile devrederek kaynak yönetimini güvenli ve öngörülebilir hale getirir.

## Temel kullanım

```java
try (BufferedReader reader = new BufferedReader(new FileReader("notes.txt"))) {
    String line = reader.readLine();
    System.out.println(line);
}
```

`try` parantezinin içinde tanımlanan her kaynak, blok normal ya da istisnayla sona erdiğinde otomatik olarak kapatılır. Kapatma sırası tanımlama sırasının tersidir: en son açılan kaynak ilk kapatılır. Bunu sağlamak için kaynağın `java.lang.AutoCloseable` arayüzünü uygulaması yeterlidir; `close()` metodunu kendiniz çağırmanıza gerek kalmaz.

## Birden fazla kaynak ve bastırılmış istisnalar

```java
try (var in = new FileInputStream("a.txt");
     var out = new FileOutputStream("b.txt")) {
    in.transferTo(out);
} catch (IOException e) {
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Bastirilmis: " + suppressed);
    }
}
```

Aynı blokta birden fazla kaynak noktalı virgülle ayrılarak açılabilir. Eğer hem gövde hem de `close()` çağrısı istisna fırlatırsa, gövdedeki istisna asıl fırlatılan olur; `close()`'dan gelen istisna kaybolmaz, `addSuppressed()` ile ana istisnaya eklenir ve `getSuppressed()` ile geri alınabilir.

## Dikkat edilmesi gerekenler

- Kaynak, `AutoCloseable` (ya da onun alt arayüzü `Closeable`) uygulamıyorsa `try-with-resources` içinde kullanılamaz.
- Java 9 itibarıyla kaynak değişkeninin `try` parantezinin içinde tanımlanması zorunlu değildir; blok dışında tanımlanmış `effectively final` bir değişken de doğrudan referans olarak verilebilir.
- Kendi sınıflarınızda `AutoCloseable` uygularken `close()` metodunu idempotent yazmak iyi bir pratiktir, çünkü bazı akışlarda birden fazla kez çağrılabilir.

## Denemek için

```
jshell> try (var r = new java.io.StringReader("test")) { System.out.println(r.read()); System.out.println(r.read()); }
```

İki `read()` çağrısının art arda `116` ve `101` (yani `t` ve `e` karakterlerinin kod noktaları) döndürdüğünü, blok bittiğinde kaynağın otomatik kapandığını görürsünüz. `r`'i blok dışında tekrar kullanmaya çalışırsanız kapsam dışı kaldığı için derleme hatası alırsınız.

[Kaynak: https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html]
