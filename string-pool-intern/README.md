# String Pool ve intern()

`String` nesneleri Java'da değişmezdir (immutable), bu yüzden JVM aynı metin içeriğini tekrar tekrar bellekte tutmak yerine paylaşabilir. Bu paylaşım mekanizmasına "string pool" (string havuzu) denir; ama hangi string'in havuzda olduğu ve `==` ile karşılaştırmanın ne zaman güvenli olduğu sık karıştırılan bir konudur.

## Literal'ler ve pool

```java
String a = "merhaba";
String b = "merhaba";
String c = new String("merhaba");

System.out.println(a == b);        // true
System.out.println(a == c);        // false
System.out.println(a.equals(c));   // true
```

Derleme zamanında bilinen string literal'ler otomatik olarak string pool'a konur; aynı içerikteki iki literal aynı referansı paylaşır, bu yüzden `a == b` `true` döner. `new String(...)` ise pool'dan bağımsız, heap üzerinde yeni bir nesne yaratır; içerik aynı olsa da referans farklıdır. İçerik karşılaştırması için her zaman `equals`, referans karşılaştırması gereken nadir durumlarda `==` kullanılmalıdır.

## intern() ile pool'a katılma

```java
String s1 = new String("kayit-" + id);
String s2 = s1.intern();

String s3 = "kayit-" + id;
System.out.println(s2 == s3.intern());  // true
```

`intern()` çağrıldığında JVM, string pool'da bu içerikle eşleşen bir referans olup olmadığına bakar; varsa onu döndürür, yoksa bu string'i pool'a ekleyip referansını döndürür. Çalışma zamanında (`+` ile birleştirme, `substring` gibi) üretilen string'ler normalde pool'a girmez; `intern()` bu string'leri de pool'a dahil ederek literal'lerle referans eşitliği kurmayı mümkün kılar.

## Dikkat edilmesi gerekenler

- `intern()` aşırı kullanıldığında pool büyür ve bu bellek (Java 7 öncesinde PermGen, sonrasında heap) üzerinde baskı yaratabilir; yalnızca gerçekten çok tekrar eden string'ler için düşünülmelidir.
- İçerik eşitliği için asla `==` kullanmayın; bu, pool davranışına bağlı kırılgan kod üretir.
- `String` dışındaki nesnelerde `intern()` benzeri bir mekanizma yoktur; bu davranış `String`'e özgüdür.

## Denemek için

```
jshell> var x = new String("test")
jshell> var y = "test"
jshell> x == y
jshell> x.intern() == y
```

İlk karşılaştırma `false` döner çünkü `x` heap'te ayrı bir nesnedir; `intern()` çağrısından sonraki karşılaştırma ise `true` döner çünkü `x.intern()` artık pool'daki `"test"` referansını işaret eder.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#intern()]
