# arrays-aslist-pitfall

`Arrays.asList(...)`, bir diziyi hızlıca `List` arayüzü üzerinden kullanmanın en kısa yoludur ve çoğu geliştirici onu sıradan bir `ArrayList` gibi davranır sanır. Oysa döndürülen nesne `java.util.Arrays` sınıfının kendi iç `ArrayList` implementasyonudur; sabit boyutludur ve verilen diziyi kopyalamak yerine ona doğrudan bir görünüm (`view`) sağlar. Bu iki fark, çalışma zamanında beklenmedik istisnalara ve sessiz veri tutarsızlıklarına yol açar.

## Sabit boyutlu liste davranışı

```java
List<String> names = Arrays.asList("ali", "veli", "ayse");
names.set(0, "mehmet"); // calisir, boyut degismiyor
names.add("fatma");     // UnsupportedOperationException
```

`set(int, E)` boyutu değiştirmediği için sorunsuz çalışır, ancak `add`, `remove` ve `clear` gibi boyutu değiştiren metotlar `UnsupportedOperationException` fırlatır. Bunun nedeni, dönen listenin `AbstractList` üzerinden yalnızca `set` ve `get`'i uygulaması, boyut değiştiren metotları ise desteklememesidir.

## Diziyle paylaşılan bellek

```java
Integer[] source = {1, 2, 3};
List<Integer> view = Arrays.asList(source);
source[0] = 99;
System.out.println(view.get(0)); // 99
```

`Arrays.asList`, diziyi kopyalamaz; döndürülen liste doğrudan aynı dizi üzerinde çalışan bir sarmalayıcıdır. Diziye yapılan bir değişiklik listeye, listeye yapılan `set` çağrısı da diziye yansır. Bu paylaşım, özellikle bir metottan dönen diziyi listeye çevirip çağrı tarafına geri verdiğinizde istenmeyen yan etkilere (`aliasing`) yol açabilir.

## Dikkat edilmesi gerekenler

- İlkel tip dizisiyle (`int[]`) kullanıldığında `Arrays.asList(intArray)`, her elemanı `Integer`'a kutulamaz; tek elemanlı bir `List<int[]>` döner. Doğru sonuç için `Integer[]` kullanılmalıdır.
- Bağımsız ve tam olarak değiştirilebilir bir liste istiyorsanız `new ArrayList<>(Arrays.asList(...))` ile kopyalayın.
- Salt okunur, hem boyut hem eleman değişikliğine tamamen kapalı bir liste için `List.of(...)` tercih edilmelidir; `Arrays.asList` yalnızca eleman değişikliğine izin verir, boyut değişikliğine izin vermez.

## Denemek için

```
jshell> var arr = new Integer[]{1, 2, 3}
jshell> var list = Arrays.asList(arr)
jshell> arr[0] = 42
jshell> list
jshell> list.add(4)
```

Son satırda `UnsupportedOperationException` alırken, `list`'in ilk elemanının diziye yapılan değişiklikle birlikte `42`'ye döndüğünü göreceksiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#asList(T...)]
