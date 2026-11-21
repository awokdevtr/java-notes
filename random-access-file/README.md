# RandomAccessFile

Normal akış tabanlı okuma/yazma (`InputStream`/`OutputStream`, `Reader`/`Writer`) bir dosyayı baştan sona sırayla gezmeyi zorunlu kılar; bir dosyanın ortasındaki tek bir kaydı güncellemek için ya tüm dosyayı yeniden yazmanız ya da karmaşık bir tampon mekanizması kurmanız gerekir. `java.io.RandomAccessFile`, dosya içinde serbestçe ileri-geri zıplayan bir imleç (`file pointer`) sunarak sabit boyutlu kayıtları yerinde okuyup değiştirmeyi mümkün kılar.

## Konuma atlayıp okuma/yazma

```java
try (RandomAccessFile raf = new RandomAccessFile("data.bin", "rw")) {
    raf.writeInt(100);
    raf.writeInt(200);
    raf.writeInt(300);

    raf.seek(4);
    raf.writeInt(999);

    raf.seek(0);
    System.out.println(raf.readInt());
    System.out.println(raf.readInt());
    System.out.println(raf.readInt());
}
```

`seek(long pos)` imleci dosyanın tam `pos`. baytına konumlandırır; her `readXxx`/`writeXxx` çağrısı bu konumdan başlar ve okunan/yazılan bayt sayısı kadar imleci ileri taşır. `"rw"` modu dosyayı yoksa oluşturur ve hem okuma hem yazma izni verir; `"r"` modunda yazma metotları `IOException` fırlatır. `writeInt`/`readInt` gibi metotlar her zaman 4 baytlık `big-endian` ikili temsil kullanır, bu yüzden platformdan bağımsız sabit boyutlu alanlar tasarlamak mümkündür.

## Sabit boyutlu kayıtlarla rastgele erişim

```java
long recordSize = 16;
long recordIndex = 5;

raf.seek(recordIndex * recordSize);
byte[] record = new byte[(int) recordSize];
raf.readFully(record);
```

Her kaydın boyutu sabit olduğunda, `kayıt_no * kayıt_boyutu` ile istenen kaydın bayt ofseti doğrudan hesaplanır ve dosyanın öncesini okumaya gerek kalmadan tek bir `seek` ile o kayda ulaşılır. `readFully` sıradan `read`'den farklı olarak istenen bayt sayısı tamamen okunana kadar bekler ve dosya sonuna gelindiğinde `EOFException` fırlatır; kısmi okumayla sessizce devam etmez.

## Dikkat edilmesi gerekenler

- `RandomAccessFile` tamponsuz (`unbuffered`) çalışır; sık ve küçük okuma/yazma yapan kodlarda her çağrı doğrudan sistem çağrısına gider, bu yüzden büyük sıralı işlemler için `BufferedInputStream`/`NIO` kanalları genelde daha hızlıdır.
- `getFilePointer()` imlecin o anki konumunu, `length()` dosyanın toplam boyutunu döndürür; `setLength()` ile dosyayı küçültüp büyütmek de mümkündür, büyütülen kısım sıfır baytlarla doldurulur.
- Sınıf thread-safe değildir; aynı `RandomAccessFile` nesnesi birden fazla thread'den eşzamanlı `seek`+`read` çağrılarıyla kullanılırsa imleç konumu yarışır ve okumalar birbirine karışır.

## Denemek için

```
jshell> import java.io.RandomAccessFile
jshell> var raf = new RandomAccessFile("test.bin", "rw")
jshell> raf.writeInt(42); raf.seek(0); raf.readInt()
jshell> raf.close()
```

`seek(0)` çağrısını kaldırıp tekrar `readInt()` çağırırsanız imlecin yazma sonrası dosya sonunda kaldığını ve `EOFException` fırlattığını görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/RandomAccessFile.html]
