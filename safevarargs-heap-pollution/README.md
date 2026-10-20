# @SafeVarargs ve Heap Pollution

Generic bir tipte `varargs` parametresi tanımlamak (`List<String>... lists`) derleyiciyi zor durumda bırakır: `varargs` arka planda bir dizi oluşturur, ama Java'da generic diziler oluşturulamaz (`type erasure` yüzünden). Derleyici bu çatışmayı, çağrı tarafında oluşturulan diziyi ham (`raw`) bir `Object[]` gibi ele alarak çözer; bu da derleme zamanında yakalanamayan bir tip güvenliği açığına, yani `heap pollution`a kapı açar.

## Heap pollution nasıl oluşur

```java
static <T> T[] riskli(T... elemanlar) {
    return elemanlar; // dizi referansını dışarı sızdırıyor
}

Object[] sizinti = riskli("a", "b"); // uyarıyla derlenir
List<String>[] listeler = riskli(List.of("x"), List.of("y"));
Object[] kirli = listeler;
kirli[0] = List.of(42); // ClassCastException burada değil!
String s = listeler[0].get(0); // ClassCastException burada patlar
```

`riskli` metodu, derleyicinin ürettiği `T[]` dizisini doğrudan `return` ederek çağırana sızdırır. Dizi çalışma zamanında gerçekte `Object[]`'tir, bu yüzden `kirli[0] = List.of(42)` satırı hiçbir hata vermeden çalışır. Asıl `ClassCastException`, bu kirlenmiş veriye `String` bekleyen bir yerden erişildiğinde, hatanın kaynağından çok uzakta ve takip etmesi zor bir noktada fırlar. Derleyici, generic `varargs` tanımlarken `unchecked generic array creation` uyarısı vererek bunu önceden haber verir.

## @SafeVarargs ile sözleşme

```java
@SafeVarargs
static <T> List<T> listeleOlustur(T... elemanlar) {
    return List.of(elemanlar); // dizi hiçbir yere sızdırılmıyor, sadece okunuyor
}
```

`@SafeVarargs`, derleyiciye "bu metot `varargs` dizisini sadece okuyor, değiştirmiyor ve dışarı sızdırmıyor" sözü verir; buna karşılık derleyici `unchecked` uyarısını bastırır. Bu anotasyon sadece `static`, `final` ya da `private` metotlarda (ve constructor'larda) kullanılabilir, çünkü bunlar `override` edilemez — bir alt sınıfın sözleşmeyi bozup diziyi sızdırması mümkün olmamalıdır. Anotasyonu, dizinin güvenli kullanıldığını kendiniz doğrulamadan eklemek uyarıyı susturur ama asıl riski ortadan kaldırmaz.

## Dikkat edilmesi gerekenler

- `@SafeVarargs` bir derleyici kontrolü tetiklemez; sadece bir geliştirici beyanıdır. Metodun gövdesini elle denetleyip dizinin `return` edilmediğinden, bir alana atanmadığından ve dışarıya geçirilmediğinden emin olmak gerekir.
- `List.of(...)`, `Arrays.asList(...)`, `Stream.of(...)` gibi JDK metotları zaten `@SafeVarargs` ile işaretlidir; kendi generic `varargs` API'nizi yazarken bu metotlara referans almak güvenli bir tasarım örneğidir.
- `@SafeVarargs` olmayan bir generic `varargs` metodunu her çağırdığınızda derleyici uyarı basar; bu uyarıları `-Xlint:varargs` ile açıkça görebilirsiniz.

## Denemek için

```
jshell> @SafeVarargs static <T> java.util.List<T> wrap(T... xs) { return java.util.List.of(xs); }
jshell> wrap("a", "b", "c")
jshell> static <T> T[] leak(T... xs) { return xs; }
jshell> Object[] arr = leak("x", "y")
```

Son iki komut, `@SafeVarargs` eklenmemiş bir metodun `unchecked` uyarısıyla derlendiğini ve döndürdüğü dizinin çalışma zamanında gerçekte `Object[]` olduğunu gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/SafeVarargs.html]
