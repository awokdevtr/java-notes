# custom-iterable-implementation

`for (Element e : koleksiyon)` sözdizimi sadece dilin hazır koleksiyon sınıflarına özel değildir; kendi sınıfınız da `for-each` döngüsünde kullanılabilir hale gelebilir. Derleyici bunu özel bir tipe değil, bir sözleşmeye göre çözer: tip `java.lang.Iterable` arayüzünü uygulamalıdır. Bu, iç veri yapınızı dışarıya sızdırmadan sırayla gezinmeyi mümkün kılar.

## Iterable ve Iterator'ı birlikte uygulamak

```java
class Sepet implements Iterable<String> {
    private final String[] urunler;
    private int boyut;

    Sepet(int kapasite) {
        urunler = new String[kapasite];
    }

    void ekle(String urun) {
        urunler[boyut++] = urun;
    }

    @Override
    public Iterator<String> iterator() {
        return new Iterator<>() {
            private int index = 0;

            @Override
            public boolean hasNext() {
                return index < boyut;
            }

            @Override
            public String next() {
                if (!hasNext()) {
                    throw new NoSuchElementException();
                }
                return urunler[index++];
            }
        };
    }
}
```

`Iterable<T>` arayüzünün tek soyut metodu `iterator()`'dır; döndürdüğü `Iterator<T>` nesnesi kendi gezinti durumunu (`index`) taşır, böylece aynı `Sepet` üzerinde eş zamanlı birden fazla `for-each` çalışsa bile her biri kendi konumunu bağımsız izler. `hasNext()` sınır kontrolünü yapar, `next()` ise sınır dışına çıkıldığında `NoSuchElementException` fırlatmakla sözleşmeye uyar.

## Derleyicinin for-each'e çevirmesi

```java
for (String urun : sepet) {
    System.out.println(urun);
}
```

Bu döngü derleme sırasında aşağıdakine eşdeğer bir bayt koduna indirilir:

```java
Iterator<String> it = sepet.iterator();
while (it.hasNext()) {
    String urun = it.next();
    System.out.println(urun);
}
```

Yani `for-each` bir dil büyüsü değil, `Iterable`/`Iterator` sözleşmesi üzerine kurulu sözdizimsel şekerdir (`syntactic sugar`). Bu da `Iterable` uygulamayan bir tipi `for-each` içinde kullanmaya çalıştığınızda neden derleme hatası aldığınızı açıklar.

## Dikkat edilmesi gerekenler

- `iterator()` her çağrıldığında *yeni* bir `Iterator` döndürmelidir; aynı iterator nesnesini paylaşmak, iç içe veya ardışık `for-each` döngülerinin birbirinin gezinti durumunu bozmasına yol açar.
- Döngü sırasında koleksiyonun yapısını değiştirmek (`boyut`'u elle değiştirmek gibi) `ConcurrentModificationException` fırlatmaz çünkü bu örnekteki iterator fail-fast kontrolü yapmıyor; kendi sınıflarınızda bu garantiyi istiyorsanız bir değişiklik sayacı (`modCount`) eklemeniz gerekir.
- `Iterable`, `remove()` desteği sunmaz; eleman kaldırmayı destekleyen bir iterator için `Iterator.remove()`'ı override etmek gerekir.

## Denemek için

```
jshell> Iterable<Integer> tekSayilar = () -> java.util.List.of(1, 3, 5).iterator()
jshell> for (int sayi : tekSayilar) { System.out.println(sayi); }
```

`Iterable`'ın tek soyut metodu olduğu için bir lambda ile de uygulanabildiğini, derleyicinin bunu `@FunctionalInterface` gibi kabul ettiğini görürsünüz. Çıktı sırasıyla `1`, `3`, `5` olur.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html]
