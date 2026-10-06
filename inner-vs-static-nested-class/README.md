# inner-vs-static-nested-class

Bir sınıfın içine başka bir sınıf tanımlamak, ilişkili yardımcı tipleri dışa taşımadan bir arada tutmanın yoludur. Ama `static` anahtar kelimesinin varlığı veya yokluğu, iç sınıfın dış sınıfın örneğine bağlı olup olmadığını tamamen değiştirir; bu fark göz ardı edildiğinde gereksiz referans tutma ve istenmeyen bellek sızıntıları ortaya çıkar.

## Static nested class: bağımsız bir iç sınıf

```java
public class Outer {
    static class Node {
        final int value;
        Node(int value) { this.value = value; }
    }

    static Node createNode(int value) {
        return new Node(value);
    }
}

Outer.Node node = new Outer.Node(42);
```

`static` ile tanımlanan iç sınıf, dış sınıfın belirli bir örneğine bağlı değildir; paket seviyesindeki bağımsız bir sınıf gibi davranır ve sadece dış sınıfın `static` üyelerine erişebilir. Örneklemek için dış sınıfın bir nesnesine gerek yoktur, `new Outer.Node(...)` şeklinde doğrudan oluşturulur. Bu yüzden `Map.Entry` gibi yardımcı veri taşıyıcıları genellikle static nested class olarak tasarlanır.

## Inner class: dış örneğe gizli referans

```java
public class Outer {
    private final int id;
    Outer(int id) { this.id = id; }

    class Inner {
        void printOuterId() {
            System.out.println(id); // Outer.this.id
        }
    }
}

Outer outer = new Outer(7);
Outer.Inner inner = outer.new Inner();
inner.printOuterId(); // 7
```

`static` olmadan tanımlanan inner class, derleyici tarafından otomatik olarak dış sınıfın örneğine bir referans (`Outer.this`) taşır; bu yüzden örneklemek için önce bir `Outer` nesnesi gerekir, `outer.new Inner()` sözdizimi bu gizli bağı somutlaştırır. Inner class nesnesi dış nesneden daha uzun ömürlü bir yerde (örneğin statik bir koleksiyonda) saklanırsa, bu gizli referans dış nesnenin `GC` tarafından toplanmasını engeller.

## Dikkat edilmesi gerekenler

- Bir iç sınıf dış sınıfın örnek durumuna ihtiyaç duymuyorsa mutlaka `static` ekleyin; bu hem gizli referans taşımasını önler hem de JVM açısından daha hafiftir.
- Inner class'ları uzun ömürlü koleksiyonlarda veya `static` alanlarda saklamak, dış nesneyi canlı tutarak istenmeyen bellek sızıntısına yol açabilir.
- Anonim sınıflar ve yerel (local) sınıflar da örtük olarak çevreleyen kapsama referans tutar; aynı dikkat onlar için de geçerlidir.

## Denemek için

```
jshell> class Outer { int id = 7; class Inner { int get() { return id; } } }
jshell> Outer o = new Outer()
jshell> Outer.Inner i = o.new Inner()
jshell> i.get()
```

`o.new Inner()` sözdiziminin zorunlu olduğunu, `new Outer.Inner()` yazmaya çalışırsanız derleme hatası aldığınızı görürsünüz; çünkü inner class örneği bir `Outer` örneğine bağlı olmadan var olamaz.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html]
