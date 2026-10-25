# scanner-nextline-pitfall

`java.util.Scanner`, konsol girdisini token bazında okur: `nextInt()`, `nextDouble()` gibi metotlar sadece sayısal token'ı tüketir, satırı sonlandıran yeni satır karakterini (`\n`) girdi akışında bırakır. Hemen ardından `nextLine()` çağrıldığında bu metot "bir satırın kalanını" okur ve akışta bekleyen o boş satır sonunu alıp boş bir `String` döndürür. Sonuç, kullanıcıdan hiçbir girdi istemeden atlanan bir satırdır ve bu, en sık rastlanan `Scanner` hatalarından biridir.

## Hatanın oluşumu

```java
Scanner sc = new Scanner(System.in);
System.out.print("Yas: ");
int yas = sc.nextInt();      // "25\n" girildi, sadece "25" tüketildi
System.out.print("Isim: ");
String isim = sc.nextLine(); // kalan "\n"i okur, isim boş string olur
```

`nextInt()` token'ı ayırırken whitespace delimiter'ını (varsayılan olarak boşluk, tab, yeni satır) tüketmez, sadece sayıyı okur. İmleç yeni satırın tam öncesinde kalır. `nextLine()` ise "imleçten bir sonraki satır sonuna kadar her şeyi" okuduğu için burada sadece o bekleyen yeni satırı görür ve boş sonuç üretir.

## Düzeltme

```java
Scanner sc = new Scanner(System.in);
System.out.print("Yas: ");
int yas = sc.nextInt();
sc.nextLine();               // bekleyen yeni satırı tüket, sonucu kullanma
System.out.print("Isim: ");
String isim = sc.nextLine();
```

Sayısal bir `next*()` çağrısından sonra `String` okumaya geçerken araya fazladan bir `nextLine()` eklemek, bekleyen yeni satırı temizler. Alternatif olarak tüm girdiyi tutarlı biçimde `nextLine()` ile okuyup değerleri kendiniz `Integer.parseInt()` gibi metotlarla dönüştürmek de bu karışıklığı tamamen ortadan kaldırır.

## Dikkat edilmesi gerekenler

- Hata sadece `nextInt()`/`nextDouble()` gibi sayısal metotlardan sonra `nextLine()` çağrıldığında ortaya çıkar; iki `nextLine()` art arda kullanıldığında bu sorun yaşanmaz.
- `hasNextLine()` ile `hasNext()` da aynı delimiter mantığına tabidir; karışık kullanım aynı şekilde yanıltıcı sonuç verebilir.
- `BufferedReader` + manuel `parse` yaklaşımı, `Scanner`'ın token/satır karışımından tamamen kaçınmak isteyenler için daha öngörülebilir bir alternatiftir.

## Denemek için

```
jshell> var sc = new java.util.Scanner("25\nAhmet\n")
jshell> sc.nextInt()
jshell> sc.nextLine()
jshell> sc.nextLine()
```

İkinci `nextLine()` çağrısının boş string döndürdüğünü, asıl "Ahmet" değerinin üçüncü çağrıyla geldiğini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Scanner.html]
