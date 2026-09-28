# Generic Type Erasure

`List<String>` ile `List<Integer>` derleme zamanında iki farklı tip gibi görünür, ama JVM ikisini de aynı `.class` dosyasıyla çalıştırır. Java generics'i derleyici düzeyinde bir tip güvenliği katmanı olarak uygular; `.class` dosyası üretilirken tip parametreleri silinir (erasure). Bu, generic'lerin `instanceof` ile ayırt edilememesi ve bazı runtime tuzaklarının kaynağı olan temel tasarım kararıdır.

## Runtime'da tip parametresinin kaybolması

```java
List<String> names = new ArrayList<>();
List<Integer> ids = new ArrayList<>();

System.out.println(names.getClass() == ids.getClass()); // true
```

Derleyici `List<String>` ve `List<Integer>` arasındaki farkı yalnızca derleme sırasında, `add`/`get` çağrılarını kontrol ederken kullanır; ürettiği bytecode'da tip parametresi `Object`'e (veya sınırlıysa üst sınıra) indirgenir. `getClass()` her iki liste için de aynı `ArrayList` sınıfını döndürür, çünkü JVM'in gördüğü tek şey ham (raw) `ArrayList` tipidir. Bu yüzden `list instanceof List<String>` yazmak derleme hatasıdır; JVM elinde böyle bir tip ayrımı tutmaz.

## Unchecked cast ve heap pollution

```java
static <T> T[] firstTwo(T... items) {
    return Arrays.copyOf(items, 2); // "unchecked" uyarısı
}

Object[] raw = firstTwo("a", "b", "c");
String[] strings = (String[]) raw; // ClassCastException riski
```

Generic varargs metotları arka planda `Object[]` üzerinde çalışır, çünkü gerçek tip parametresi bytecode'da mevcut değildir; derleyici bu yüzden `Arrays.copyOf` gibi çağrılarda "unchecked" uyarısı verir. Diziyi daha spesifik bir tipe cast etmek derlenir ama garantisi yoktur: çalışma zamanında beklenmedik bir eleman tipiyle karşılaşılırsa `ClassCastException` fırlar. Bu duruma "heap pollution" denir ve generic ile dizi kovaryantlığının (array covariance) bir araya gelmesinden doğar.

## Dikkat edilmesi gerekenler

- `T.class` veya `new T()` yazamazsınız; tip parametresi runtime'da bilinmediği için bu ifadeler derlenmez.
- Aynı ham tipi paylaşan iki generic metodu overload edemezsiniz (örn. `void m(List<String> l)` ve `void m(List<Integer> l)` aynı anda tanımlanamaz).
- `@SafeVarargs`, yalnızca metodun varargs parametresini güvenli kullandığınızı derleyiciye bildirir; erasure'ın kendisini ortadan kaldırmaz, sadece uyarıyı bastırır.

## Denemek için

```
jshell> List<String> a = new ArrayList<>()
jshell> List<Integer> b = new ArrayList<>()
jshell> a.getClass() == b.getClass()
jshell> a.getClass().getTypeParameters().length
```

İlk karşılaştırma `true` döner çünkü ikisi de aynı `ArrayList` sınıfına aittir; `getTypeParameters()` ise sınıfın kendi bildirdiği tip parametresi sayısını gösterir, örneğin bir listenin hangi somut tiple oluşturulduğunu değil.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/generics/erasure.html]
