# instance-initializer-block

Constructor'lar arasında tekrar eden başlatma kodunu tek yere toplamanın bir yolu `static` olmayan bir `{}` bloğudur: sınıf gövdesine doğrudan yazılan bu blok, her nesne oluşturulduğunda çalışır ve birden fazla constructor varsa ortak kurulum mantığını tek bir yerde tutmanıza izin verir. Az bilindiği için genelde ya fark edilmez ya da `static` bloklarla karıştırılır, ama derleyici onu her constructor'ın başına örtük olarak kopyalar.

## Çalışma zamanı ve sırası

```java
class Session {
    private final String id;
    private int requestCount;

    {
        requestCount = 1;
        System.out.println("instance blok calisti, id henuz: " + id);
    }

    Session() {
        this("anon");
    }

    Session(String id) {
        this.id = id;
        System.out.println("constructor govdesi, id=" + id);
    }
}
```

Instance initializer blok, alan başlatıcılarıyla aynı sırada ve seçilen constructor'ın `this()`/`super()` çağrısından sonra, ama kendi gövdesinden önce çalışır. `new Session()` çağrısında önce `Session(String)` seçilir, `super()` örtük olarak çalışır, ardından blok (`requestCount = 1`, henüz atanmamış `id` için `null` yazdırır) işletilir, en son `Session(String)` gövdesi çalışıp `id`'yi atar. Birden fazla constructor varsa bile blok her çağrıda bir kez daha çalışır; bu yüzden `static` bloktan farklı olarak sınıf başına değil nesne başına tekrarlanır.

## Anonim sınıflarda tek kullanım alanı

```java
List<String> names = new ArrayList<>() {
    {
        add("ali");
        add("veli");
    }
};
```

Anonim sınıflarda constructor tanımlanamadığı için instance initializer blok, nesneyi oluştururken ek kurulum yapmanın tek yoludur; burada `ArrayList` alt sınıfı yaratılıp blok içinde doğrudan doldurulur. Bu kalıp pratikte "double brace initialization" olarak bilinir, ancak her kullanım gizli bir iç sınıf yarattığından `equals`/serileştirme gibi yerlerde beklenmeyen davranışlara yol açabilir.

## Dikkat edilmesi gerekenler

- Blok, kendisinden önce tanımlanmış alanlara erişebilir; sonra tanımlanan bir alana referans vermek `forward reference` derleme hatası verir.
- Birden fazla instance initializer blok olabilir; hepsi alan başlatıcılarıyla birlikte, kaynak koddaki sırayla tek bir başlatma zinciri gibi birleştirilir.
- "Double brace initialization" her kullanımda isimsiz bir alt sınıf ürettiğinden gereksiz sınıf şişmesine ve dış sınıfa gizli referans tutmaya (iç sınıf `static` değilse) yol açar; günümüzde `List.of`/`Map.of` gibi fabrika metotları tercih edilir.

## Denemek için

```
jshell> class X { { System.out.println("blok"); } X() { System.out.println("ctor"); } }
jshell> new X()
```

Çıktıda önce `blok`, sonra `ctor` görürsünüz; bu da instance initializer bloğun constructor gövdesinden önce, ama `super()` çağrısından sonra çalıştığını doğrular.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.6]
