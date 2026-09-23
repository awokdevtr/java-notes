# immutability-and-final

Paylaşılan bir nesnenin bir thread tarafından değiştirilmesi, başka bir thread'in onu okurken tutarsız veya yarı güncellenmiş bir durum görmesine yol açabilir. Bu tür hatalar kilitlemeyle (`synchronized`) çözülebilir, ama daha basit bir yaklaşım var: nesneyi hiç değiştirilemez yapmak. `final` anahtar kelimesi, tek başına yeterli olmasa da, değişmez (`immutable`) sınıflar kurmanın temel yapı taşıdır.

## final ile alan bağlama

```java
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }
}
```

`final` bir alan yalnızca bir kez, ya tanımlandığı yerde ya da constructor içinde atanabilir; sonrasında değeri değiştirilemez. Sınıfın kendisini de `final` yapmak, alt sınıfların `getX()` gibi metotları override ederek mutable davranış eklemesini engeller. Bu iki `final` kullanımı birlikte, `Point` nesnelerinin oluşturulduktan sonra asla değişmeyeceğini garanti eder.

## final yeterli değildir: derin değişmezlik

```java
public final class Team {
    private final List<String> members;

    public Team(List<String> members) {
        this.members = List.copyOf(members);
    }

    public List<String> getMembers() {
        return members;
    }
}
```

`final` yalnızca referansın kendisini sabitler, referansın gösterdiği nesnenin iç durumunu değil. Constructor'a verilen `members` listesi dışarıda hâlâ değiştirilebilir olduğundan, referansı doğrudan saklamak sızıntıya yol açar. `List.copyOf()` ile savunmacı bir kopya almak ve `getMembers()`'dan da mutable bir referans döndürmemek, gerçek anlamda değişmezlik için gereklidir.

## Ne zaman işe yarar

- Thread'ler arası paylaşılan veriyi kilitlemeden güvenle paylaşmak istediğinizde.
- `HashMap` anahtarı gibi hash koduna güvenilen nesnelerde: değişmezlik, nesne koleksiyona eklendikten sonra hash kodunun sabit kalmasını garanti eder.
- API sınırlarında, çağıranın elindeki nesneyi değiştirerek sizin iç durumunuzu bozmasını istemediğinizde.

## Denemek için

```
jshell> var p = new java.awt.Point(1, 2)
jshell> p.x = 5
jshell> System.out.println(p.x)
```

`java.awt.Point` mutable olduğu için `p.x = 5` derlenir ve `5` yazdırır; yukarıdaki `Point` örneğinde ise `x` alanı `final` olduğundan aynı atama derleme hatası verir. Bu fark, `final` alanların koruduğu garantiyi doğrudan gösterir.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/data/exercise1.html]
