# static-initialization-order

Bir sınıf ilk kez kullanıldığında `static` alanlar ve `static` bloklar hangi sırayla çalışır sorusu, özellikle miras hiyerarşisinde ya da birbirine bağımlı `static` alanlar olduğunda kafa karıştırır. Sıra rastgele değildir: JVM, sınıf yüklenirken kaynak koddaki tanım sırasına ve sınıf hiyerarşisine göre kesin bir çalışma planı izler. Bu sırayı bilmemek, `null` ya da beklenmeyen varsayılan değerlerle karşılaşmaya yol açabilir.

## Aynı sınıf içinde sıralama

```java
class Config {
    static int base = 10;

    static {
        System.out.println("blok calisiyor, base=" + base);
        base += 5;
    }

    static int limit = base * 2;
}
```

`static` alanlar ve `static` bloklar, sınıf içinde yazıldıkları sırayla yukarıdan aşağı çalıştırılır; blok, yalnızca kendinden önce tanımlanmış alanları görebilir. Bu örnekte `base` önce `10` olur, blok çalışıp `15`e çıkarır, ardından `limit` bu güncel değeri kullanarak `30` olarak hesaplanır. `limit`'i bloktan önce tanımlasaydınız, blok içinde ona erişmek derleme hatası verirdi çünkü henüz `forward reference` durumunda olurdu.

## Miras hiyerarşisinde sıralama

```java
class Parent {
    static { System.out.println("Parent statik blok"); }
}

class Child extends Parent {
    static { System.out.println("Child statik blok"); }

    public static void main(String[] args) {
        System.out.println("main basliyor");
    }
}
```

Bir sınıf ilk kez kullanılmadan önce JVM önce üst sınıfın `static` başlatmasını tamamlar, sonra alt sınıfınkini çalıştırır. Bu yüzden çıktı sırasıyla `Parent statik blok`, `Child statik blok`, `main basliyor` olur; `main` metodunun kendisi `Child`'da tanımlı olsa bile hiyerarşi baştan aşağı işletilir. Statik başlatma her sınıf için yalnızca bir kez, sınıf ilk yüklendiğinde gerçekleşir; sonraki nesne oluşturmalarında tekrarlanmaz.

## Dikkat edilmesi gerekenler

- Statik başlatma, sınıfa ilk erişimde (nesne oluşturma, statik alan/metot çağrısı) tetiklenir; sınıfın var olması tek başına yeterli değildir.
- Birden fazla `static` blok olabilir; hepsi tek bir başlatıcı gibi birleştirilip sırayla çalışır.
- Statik alanlar arasında dairesel bağımlılık varsa, henüz hesaplanmamış bir alan geçici olarak varsayılan değerinde (`0`, `null`) okunabilir; bu durum `NullPointerException` gibi hatalara yol açabilir.

## Denemek için

```
jshell> class A { static { System.out.println("A"); } }
jshell> class B extends A { static { System.out.println("B"); } }
jshell> new B()
```

`new B()` satırı çalıştığında önce `A`, sonra `B` yazdığını görürsünüz; bu da alt sınıfın statik başlatmasından önce üst sınıfınkinin garanti altında tamamlandığını gösterir.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.4]
