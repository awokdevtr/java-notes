# Externalizable ile serileştirme kontrolünü ele alma

`Serializable` bir sınıfı işaretlemek yeterlidir; JVM, reflection kullanarak her alanı otomatik olarak yazar/okur ve bu süreç geliştiricinin kontrolü dışındadır. Hangi byte'ların diske/ağa gittiğini birebir belirlemek, varsayılan serileştirmenin reflection maliyetinden kaçınmak ya da sınıf alanlarını değiştirmeden format sürümlemek gerektiğinde `java.io.Externalizable` devreye girer: `writeExternal`/`readExternal` metotlarıyla serileştirmenin tamamı elle yazılır.

## writeExternal / readExternal ile elle serileştirme

```java
public class Point implements Externalizable {
    private int x;
    private int y;

    public Point() {} // zorunlu: no-arg constructor

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeInt(x);
        out.writeInt(y);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException {
        x = in.readInt();
        y = in.readInt();
    }
}
```

`Externalizable`, `Serializable`'ı genişletir ama `writeObject`/`readObject` yerine iki `public` metot zorunlu kılar; bu metotlar hangi alanın hangi sırayla yazılacağını tam olarak belirler, dolayısıyla reflection ile alan taraması yapılmaz ve format tamamen geliştiricinin elindedir. Deserializasyon sırasında JVM, nesneyi alanları doldurmadan önce no-arg constructor ile oluşturur; bu yüzden `public` bir no-arg constructor olmadan `InvalidClassException` fırlatılır — varsayılan `Serializable` akışında bu kısıtlama yoktur.

## Sürüm esnekliği ve sıralama kontrolü

```java
@Override
public void writeExternal(ObjectOutput out) throws IOException {
    out.writeInt(1);       // format versiyonu
    out.writeUTF(name);
    out.writeInt(x);
}

@Override
public void readExternal(ObjectInput in) throws IOException {
    int version = in.readInt();
    name = in.readUTF();
    x = (version >= 1) ? in.readInt() : 0;
}
```

Yazma/okuma sırası elle kodlandığı için yeni bir alan eklendiğinde, akışa açıkça bir versiyon numarası koyup `readExternal` içinde buna göre dallanmak mümkündür; `serialVersionUID` tabanlı varsayılan uyumluluk kontrolüne bağlı kalınmaz. Bu esneklik bedelsiz değildir: alan ekleme/çıkarma/sıralama değişikliği artık derleyici tarafından değil tamamen bu iki metodun tutarlılığıyla garanti edilir, bir satır unutulursa veri sessizce bozulur.

## Dikkat edilmesi gerekenler

- No-arg constructor `public` olmalıdır; `private`/`protected` ya da eksik olması deserializasyonu `InvalidClassException` ile başarısız kılar.
- `transient` anahtar kelimesi `Externalizable` için anlamsızdır — hangi alanın yazılacağına JVM değil `writeExternal` içindeki kod karar verir.
- `super.readExternal`/`writeExternal` çağrısı otomatik yapılmaz; kalıtım zincirinde üst sınıfın durumu varsa bunu elle çağırmak geliştiricinin sorumluluğundadır.

## Denemek için

```
jshell> import java.io.*
jshell> record P() {}  // gerçek örnek için yukarıdaki Point sınıfını bir .java dosyasına yazıp derleyin
jshell> var baos = new ByteArrayOutputStream()
jshell> var oos = new ObjectOutputStream(baos)
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Externalizable.html]
