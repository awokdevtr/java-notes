# Statik Metot Gizleme (Method Hiding)

Bir alt sınıf, üst sınıftaki bir `static` metotla aynı imzada kendi `static` metodunu tanımladığında bu bir `override` değil, `hiding`'dir (gizleme). `static` metotlar polimorfik değildir: hangi metodun çağrılacağına derleyici, referansın **derleme zamanı tipine** bakarak karar verir, nesnenin çalışma zamanındaki gerçek tipine değil. Bu fark bilinmediğinde, üst sınıf tipiyle tutulan bir referans üzerinden çağrılan `static` metodun "yanlış" sınıftan çalıştığı izlenimi doğar.

## Statik metotta gizleme

```java
class Animal {
    static String sesVer() { return "Hayvan sesi"; }
}

class Kedi extends Animal {
    static String sesVer() { return "Miyav"; }
}

Animal a = new Kedi();
System.out.println(a.sesVer()); // "Hayvan sesi" basılır
```

`a` değişkeni çalışma zamanında bir `Kedi` nesnesini gösterse de, derleme zamanı tipi `Animal` olduğu için derleyici `sesVer()` çağrısını doğrudan `Animal.sesVer()`'e bağlar. Bu bağlama, kodun çalıştığı JVM'den ya da nesnenin gerçek tipinden tamamen bağımsız, salt derleme zamanı kararıdır; `static` çağrı aslında `Animal.sesVer()` yazmakla eşdeğerdir, sadece örnek referansı üzerinden yazılmıştır.

## Instance metotta override ile karşılaştırma

```java
class Animal {
    String sesVer() { return "Hayvan sesi"; }
}

class Kedi extends Animal {
    @Override
    String sesVer() { return "Miyav"; }
}

Animal a = new Kedi();
System.out.println(a.sesVer()); // "Miyav" basılır
```

Aynı senaryoda metot `static` olmadığında JVM, çağrıyı `a`'nın referans tipine değil nesnenin gerçek çalışma zamanı tipine göre `virtual method dispatch` ile çözer; bu yüzden `Kedi.sesVer()` çalışır. `static` gizleme ile `instance` `override` arasındaki bu ayrım, Java'nın polimorfizminin yalnızca `instance` metotları kapsadığını, `static` üyeleri kapsamadığını gösterir.

## Dikkat edilmesi gerekenler

- `static` bir metodu `@Override` ile işaretlemek derleme hatası verir; derleyici bunun bir `override` değil `hiding` olduğunu bilir ve yanlış kullanım olarak reddeder.
- `static` metotları her zaman sınıf adı üzerinden çağırmak (`Animal.sesVer()`) niyeti netleştirir; bir örnek referansı üzerinden çağırmak (`a.sesVer()`) yanıltıcıdır ve IDE'ler genelde bunu uyarı olarak işaretler.
- `private` metotlar da alt sınıfta miras alınmadığından benzer şekilde `override` edilmez; aynı imzayla tanımlanan alt sınıf metodu, tamamen bağımsız yeni bir metottur.
- Alanlar (`field`) da `static` metotlar gibi derleme zamanı tipine göre çözülür; bu yüzden alt sınıfta aynı isimde bir alan tanımlamak da benzer bir gizleme tuzağı yaratır.

## Denemek için

```
jshell> class Animal { static String ses() { return "Hayvan"; } }
jshell> class Kedi extends Animal { static String ses() { return "Miyav"; } }
jshell> Animal a = new Kedi()
jshell> a.ses()
```

Çıktı olarak "Hayvan" göreceksiniz; `Kedi.ses()`'i çağırmak için ya `((Kedi) a).ses()` cast'i ya da doğrudan `Kedi.ses()` yazmak gerekir, çünkü `static` çağrı hiçbir zaman nesnenin gerçek tipine bakmaz.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4.8.2]
