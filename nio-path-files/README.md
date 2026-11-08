# NIO.2: Path ve Files

Eski `java.io.File` API'si dosya işlemlerinde `boolean` dönen metotlara (`delete()`, `mkdir()`, `renameTo()`) dayanır; işlem neden başarısız oldu bilgisi kaybolur, sadece `false` döner. Java 7 ile gelen NIO.2, `Path` ve `Files` sınıflarıyla bu işlemleri `IOException` fırlatan, nedeni taşıyan ve dosya sistemiyle daha doğrudan konuşan bir API'ye taşır.

## Path oluşturma ve birleştirme

```java
import java.nio.file.Path;

Path base = Path.of("/var/data");
Path config = base.resolve("app/config.json");
Path normalized = Path.of("/var/data/app/../app/config.json").normalize();

System.out.println(config);      // /var/data/app/config.json
System.out.println(normalized);  // /var/data/app/config.json
System.out.println(config.getFileName());  // config.json
System.out.println(config.getParent());    // /var/data/app
```

`Path.of` hiçbir dosya sistemi erişimi yapmadan sadece soyut bir yol temsili oluşturur; dosyanın var olup olmaması önemsizdir. `resolve` iki yolu dosya sistemine özgü ayraç kurallarına göre birleştirir, `normalize` ise `.`/`..` bileşenlerini eleyerek yolu sadeleştirir. Bu işlemler tamamen bellek içinde string manipülasyonudur, henüz diske dokunulmaz.

## Files ile okuma, yazma ve kontrol

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.io.IOException;

Path file = Path.of("notes.txt");
try {
    Files.writeString(file, "ilk satir\n");
    if (Files.exists(file) && Files.isReadable(file)) {
        String content = Files.readString(file);
        System.out.println(content);
    }
} catch (IOException e) {
    System.err.println("dosya islemi basarisiz: " + e.getMessage());
}
```

`Files.writeString`/`Files.readString` tüm dosyayı tek seferde yazıp okur ve her ikisi de `IOException` fırlatır; eski `File.createNewFile()`'ın aksine başarısızlığın nedeni (izin yok, disk dolu, dizin eksik) exception mesajında gelir. `Files.exists`/`Files.isReadable` ise dosya sistemiyle anlık bir kontrol yapar ve hiçbir istisna fırlatmaz, bu yüzden "önce kontrol et sonra işlem yap" akışlarında güvenle kullanılabilir.

## Dikkat edilmesi gerekenler

- `Path` değişmezdir (`immutable`); `resolve`/`normalize` orijinal `Path`'i değiştirmez, yeni bir `Path` döndürür.
- `Files.exists` ile kontrol edip hemen ardından `Files.readString` çağırmak arasında başka bir işlem dosyayı silebilir (TOCTOU); kritik kodda kontrolü atlayıp doğrudan işlemi deneyip `IOException`'ı yakalamak daha güvenlidir.
- `Path.of("a", "b", "c")` platforma göre doğru ayraçla (`/` veya `\`) birleştirir; string'leri elle `+` ile birleştirmek taşınabilirliği bozar.

## Denemek için

```
jshell> import java.nio.file.*
jshell> var p = Files.writeString(Path.of("/tmp/demo.txt"), "merhaba")
jshell> Files.readString(p)
jshell> Files.exists(Path.of("/tmp/olmayan.txt"))
```

İlk satır dosyayı oluşturup `Path`'i döndürür, ikinci satır içeriği okur, üçüncü satır var olmayan bir dosya için exception fırlatmadan sadece `false` döner.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html]
