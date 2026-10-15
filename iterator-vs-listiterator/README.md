# Iterator vs ListIterator

`Iterator`, bir koleksiyon üzerinde yalnızca ileri yönde gezinme ve `remove()` ile eleman
kaldırma imkanı sunar. Geriye doğru gitmek, gezinti sırasında yeni eleman eklemek ya da
mevcut elemanı yerinde değiştirmek gerektiğinde bu arayüz yetersiz kalır. `List` arayüzü,
tam olarak bu ihtiyaç için `Iterator`'ı genişleten `ListIterator`'ı sağlar.

## Çift yönlü gezinme ve yerinde değiştirme

```java
List<String> names = new ArrayList<>(List.of("ada", "linus", "grace"));
ListIterator<String> it = names.listIterator();

while (it.hasNext()) {
    String name = it.next();
    if (name.equals("linus")) {
        it.set(name.toUpperCase());
    }
}

while (it.hasPrevious()) {
    System.out.println(it.previousIndex() + ": " + it.previous());
}
```

`hasPrevious()`/`previous()` çifti, cursoru listenin sonundan başına doğru gezdirir;
`previousIndex()`/`nextIndex()` cursorun o anki konumunu verir. `set()` ise son
`next()`/`previous()` çağrısının döndürdüğü elemanı, yapısal boyutu değiştirmeden
yerinde günceller. Bu metotlar sade `Iterator`'da yoktur, çünkü `Iterator` sırayı veya
indeksi bilen bir yapı varsaymaz.

## Gezinti sırasında eleman ekleme

```java
ListIterator<Integer> nums = new ArrayList<>(List.of(1, 2, 3)).listIterator();
while (nums.hasNext()) {
    int n = nums.next();
    if (n == 2) {
        nums.add(99);
    }
}
// Liste artık: [1, 2, 99, 3]
```

`add()`, cursorun o anki konumuna hemen önce bir eleman yerleştirir ve cursoru yeni
elemanın sonrasına taşır; bir sonraki `next()` çağrısı eklenen elemanı atlayıp ondan
sonrakini döndürür. Bu değişiklik aynı `ListIterator` üzerinden yapıldığı için
`ConcurrentModificationException` fırlatılmaz — düz `Iterator` ile aynı işi yapmaya
çalışmak (listeyi doğrudan değiştirmek) tam olarak bu istisnaya yol açar.

## Ne zaman işe yarar

- Listeyi geriye doğru gezmek gerektiğinde
- Gezinti sırasında eleman eklemek veya mevcut elemanı değiştirmek gerektiğinde
- Cursorun indeks bilgisine ihtiyaç duyulduğunda

## Dikkat edilmesi gerekenler

- `ListIterator`, yalnızca `List` implementasyonlarında mevcuttur; `Set` veya `Map` böyle
  bir gezinti sunmaz çünkü sıralı indeks kavramları yoktur
- Hâlâ fail-fast'tir: liste başka bir yoldan (örneğin doğrudan `list.add()`) değiştirilirse
  bir sonraki `next()`/`previous()` çağrısı yine `ConcurrentModificationException` fırlatır
- `set()`, yalnızca son `next()`/`previous()` çağrısından sonra ve araya `add()`/`remove()`
  girmemişse geçerlidir; aksi halde `IllegalStateException` fırlatılır

## Denemek için

```
jshell> var l = new java.util.ArrayList<>(java.util.List.of("a","b","c"))
jshell> var it = l.listIterator()
jshell> while (it.hasNext()) { var v = it.next(); if (v.equals("b")) it.set("B"); }
jshell> l
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html]
